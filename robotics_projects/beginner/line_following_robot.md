# 🤖 High-Speed Line Following Robot with PD Control

<div align="center">

[ ⬅️ Back to Projects Directory ](../README.md) &nbsp;•&nbsp; [ 🏠 Hub Home ](../../README.md)

</div>

---

## 📌 Project Overview
A classic yet rigorous entry point into mechatronics and closed-loop control. Rather than relying on naive binary `if-else` thresholding (which produces violent zig-zag oscillations and low top speeds), this project implements an **analog weighted error sensor array** driven by a continuous **Proportional-Derivative (PD) feedback controller**.

```mermaid
graph LR
    IR["5-Channel IR Reflectance Array<br/>(Analog Ground Reflection)"] --> ADC["Weighted Average Error Calculation<br/>e(t) ∈ [-4.0, +4.0]"]
    ADC --> PD["PD Controller<br/>u(t) = Kp * e(t) + Kd * de/dt"]
    PD --> PWM["Differential Motor Mixing<br/>Left: BaseSpeed - u(t)<br/>Right: BaseSpeed + u(t)"]
    PWM --> DRV["H-Bridge Driver (TB6612FNG)"]
    DRV --> MOT["Dual DC Gearmotors (N20)"]

    style IR fill:#1e1e2e,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4
    style ADC fill:#181825,stroke:#a6e3a1,stroke-width:2px,color:#cdd6f4
    style PD fill:#11111b,stroke:#f9e2af,stroke-width:2px,color:#cdd6f4
    style PWM fill:#1e1e2e,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4
    style DRV fill:#181825,stroke:#cba6f7,stroke-width:2px,color:#cdd6f4
    style MOT fill:#11111b,stroke:#89dceb,stroke-width:2px,color:#cdd6f4
```

---

## 📋 Bill of Materials (BOM)

| Component | Specification | Quantity | Voltage / Interface | Est. Cost (USD) |
| :--- | :--- | :---: | :--- | :---: |
| **Microcontroller** | Arduino Nano (ATmega328P) | 1 | 5V / USB / 8x ADC pins | \$4.00 |
| **Sensor Array** | QTR-8A or 5-Channel TCRT5000 IR Array | 1 | 5V / Analog outputs | \$6.00 |
| **Motor Driver** | TB6612FNG Dual MOSFET H-Bridge | 1 | Logic: 5V, Motor: 6–12V | \$3.50 |
| **Actuators** | N20 Micro Metal Gearmotors (6V, 600 RPM) | 2 | 6V DC PWM | \$8.00 |
| **Wheels** | Silicone High-Traction Wheels (34mm) | 2 | D-shaft mount | \$3.00 |
| **Chassis** | Laser-cut Acrylic or 3D Printed PETG Base | 1 | Lightweight frame + caster ball | \$5.00 |
| **Power Source** | 2S LiPo Battery (7.4V, 800–1000mAh, 25C) | 1 | 7.4V (2S JST/XT30) | \$10.00 |
| **Voltage Regulator**| LM2596 or Mini 5V Buck Converter | 1 | Input 7.4V $\to$ Output 5.0V clean | \$2.00 |

---

## ⚡ Electrical Wiring Pinout

```
Arduino Nano Pin     Connected Component & Function
───────────────────────────────────────────────────────────────────
5V (Buck output)     VCC of IR Array, TB6612 VCC logic
GND                  Common Ground (Battery (-), Buck, Driver, Sensors)
A0                   IR Sensor 1 (Far Left, weight = -2)
A1                   IR Sensor 2 (Mid Left, weight = -1)
A2                   IR Sensor 3 (Center, weight = 0)
A3                   IR Sensor 4 (Mid Right, weight = +1)
A4                   IR Sensor 5 (Far Right, weight = +2)
D5 (PWM)             TB6612 PWMA (Left Motor Speed)
D6 (PWM)             TB6612 PWMB (Right Motor Speed)
D7                   TB6612 AIN1 (Left Motor Direction)
D8                   TB6612 AIN2 (Left Motor Direction)
D9                   TB6612 BIN1 (Right Motor Direction)
D10                  TB6612 BIN2 (Right Motor Direction)
D11                  TB6612 STBY (Standby, HIGH to enable)
VIN (Battery +)      TB6612 VM (Motor Voltage Rail, 7.4V)
```

---

## 💻 Complete Arduino Firmware (C++)

```cpp
/**
 * High-Speed Line Follower with PD Control
 * Platform: Arduino Nano (ATmega328P) + TB6612FNG Driver
 */

// Motor Driver Pins
const int PIN_PWMA = 5;
const int PIN_PWMB = 6;
const int PIN_AIN1 = 7;
const int PIN_AIN2 = 8;
const int PIN_BIN1 = 9;
const int PIN_BIN2 = 10;
const int PIN_STBY = 11;

// Analog IR Sensor Pins
const int SENSOR_PINS[5] = {A0, A1, A2, A3, A4};
const float SENSOR_WEIGHTS[5] = {-2.0, -1.0, 0.0, 1.0, 2.0};

// Calibration Thresholds (Black line vs White surface)
int sensor_min[5] = {1023, 1023, 1023, 1023, 1023};
int sensor_max[5] = {0, 0, 0, 0, 0};

// PD Control Gains
const float Kp = 32.0;    // Proportional gain (corrects instantaneous heading)
const float Kd = 18.0;    // Derivative gain (damps oscillation & handles sharp bends)
const int BASE_SPEED = 160; // Base cruise PWM (0 - 255)
const int MAX_SPEED = 240;

float last_error = 0.0;

void set_motors(int speed_left, int speed_right) {
    speed_left = constrain(speed_left, -MAX_SPEED, MAX_SPEED);
    speed_right = constrain(speed_right, -MAX_SPEED, MAX_SPEED);

    // Left Motor Direction
    digitalWrite(PIN_AIN1, speed_left >= 0 ? HIGH : LOW);
    digitalWrite(PIN_AIN2, speed_left >= 0 ? LOW : HIGH);
    analogWrite(PIN_PWMA, abs(speed_left));

    // Right Motor Direction
    digitalWrite(PIN_BIN1, speed_right >= 0 ? HIGH : LOW);
    digitalWrite(PIN_BIN2, speed_right >= 0 ? LOW : HIGH);
    analogWrite(PIN_PWMB, abs(speed_right));
}

float read_line_error() {
    float weighted_sum = 0.0;
    float total_active = 0.0;
    bool line_detected = false;

    for (int i = 0; i < 5; i++) {
        int raw = analogRead(SENSOR_PINS[i]);
        // Normalize reading from 0 (white) to 1000 (black)
        int val = map(raw, sensor_min[i], sensor_max[i], 0, 1000);
        val = constrain(val, 0, 1000);

        if (val > 250) { // Detection threshold
            line_detected = true;
            weighted_sum += val * SENSOR_WEIGHTS[i];
            total_active += val;
        }
    }

    // If completely off the line, steer hard in the direction of the last known error
    if (!line_detected || total_active == 0) {
        return (last_error > 0) ? 2.5 : -2.5;
    }

    return weighted_sum / total_active;
}

void setup() {
    pinMode(PIN_PWMA, OUTPUT);
    pinMode(PIN_PWMB, OUTPUT);
    pinMode(PIN_AIN1, OUTPUT);
    pinMode(PIN_AIN2, OUTPUT);
    pinMode(PIN_BIN1, OUTPUT);
    pinMode(PIN_BIN2, OUTPUT);
    pinMode(PIN_STBY, OUTPUT);
    digitalWrite(PIN_STBY, HIGH); // Enable driver

    // Automatic 3-second calibration: sweep sensors over black line
    unsigned long start_time = millis();
    while (millis() - start_time < 3000) {
        for (int i = 0; i < 5; i++) {
            int v = analogRead(SENSOR_PINS[i]);
            if (v < sensor_min[i]) sensor_min[i] = v;
            if (v > sensor_max[i]) sensor_max[i] = v;
        }
    }
}

void loop() {
    float error = read_line_error();
    float derivative = error - last_error;
    last_error = error;

    // Calculate PD control effort
    float control_effort = (Kp * error) + (Kd * derivative);

    int left_speed = BASE_SPEED - (int)control_effort;
    int right_speed = BASE_SPEED + (int)control_effort;

    set_motors(left_speed, right_speed);
    delay(5); // ~200 Hz control loop
}
```

---

## 🔧 Tuning & Troubleshooting Guide
1. **Calibrate First:** The robot must scan both the matte black tape and the white background during the first 3 seconds of boot to calibrate dynamic sensor thresholds.
2. **Tune $K_p$:** Start with $K_d = 0$. Increase $K_p$ until the robot follows mild curves but exhibits continuous fish-tailing oscillations on straights.
3. **Tune $K_d$:** Slowly increase $K_d$ to damp out the fish-tailing. If $K_d$ is too high, the robot will chatter and vibrate violently due to noise amplification.
4. **Caster Height:** Ensure the front IR array sits exactly **3 mm to 5 mm** above the surface. Greater distances will result in poor optical contrast.
