# 🎯 Pan-Tilt 2-DoF Vision Tracking Turret

<div align="center">

[ ⬅️ Back to Projects Directory ](../README.md) &nbsp;•&nbsp; [ 🏠 Hub Home ](../../README.md)

</div>

---

## 📌 Project Overview
A closed-loop **Visual Servoing** gimbal system. A host computer processes live video frames from a USB webcam using OpenCV, detects a target (face or colored sphere), computes the Cartesian pixel deviation from the optical frame center, and transmits corrective angular adjustments over USB serial to an Arduino controlling a 2-DoF pan-tilt servo mechanism.

```mermaid
graph LR
    CAM["USB Web Camera (640x480)"] --> CV["OpenCV Perception Pipeline<br/>(Color Thresholding / Haar Cascade)"]
    CV --> ERR["Pixel Error Calculation<br/>e_x = x_target - 320<br/>e_y = y_target - 240"]
    ERR --> SER["Serial Packet Formatting<br/>'<pan_angle, tilt_angle>\n'"]
    SER -->|USB Serial @ 115200 baud| MCU["Arduino / ESP32 Controller"]
    MCU --> SERVOS["2x Servomotors (Pan & Tilt)"]

    style CAM fill:#181825,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4
    style CV fill:#1e1e2e,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4
    style ERR fill:#11111b,stroke:#f9e2af,stroke-width:2px,color:#cdd6f4
    style MCU fill:#181825,stroke:#a6e3a1,stroke-width:2px,color:#cdd6f4
    style SERVOS fill:#1e1e2e,stroke:#cba6f7,stroke-width:2px,color:#cdd6f4
```

---

## 📋 Bill of Materials (BOM)

| Component | Specification | Quantity | Interface / Voltage | Est. Cost (USD) |
| :--- | :--- | :---: | :--- | :---: |
| **Gimbal Bracket** | 2-DoF Pan-Tilt Nylon/Aluminum Bracket | 1 | Hardware bracket set | \$4.00 |
| **Servomotors** | TowerPro MG996R (Metal Gear) or SG90 | 2 | 5V PWM (50 Hz) | \$8.00 |
| **Microcontroller** | Arduino Uno or Nano | 1 | USB Serial / 5V | \$4.00 |
| **Camera** | 1080p / 720p USB Web Camera | 1 | USB 2.0 Video Class (UVC) | \$12.00 |
| **External Power** | 5V 2A DC Power Adapter | 1 | Dedicated servo power | \$5.00 |

---

## ⚡ Electrical Wiring Pinout

```
Arduino Pin       Connected Component
────────────────────────────────────────────────────
Pin 9 (PWM)       Pan Servo Signal (Azimuth rotation)
Pin 10 (PWM)      Tilt Servo Signal (Elevation rotation)
5V Ext Power (+)  Pan Servo VCC & Tilt Servo VCC (Do NOT power from Arduino 5V!)
GND Ext Power (-) Common Ground (Must be tied to Arduino GND)
```

> [!WARNING]
> Never power moving servos directly from the Arduino's onboard 5V pin! Servos draw 500mA–1.5A stall current, which will instantly brown out the microcontroller. Always use an external 5V supply with grounds tied together.

---

## 💻 1. Host Computer Vision Tracking Script (Python)

```python
"""
Real-Time Color Tracking & Visual Servoing Client
Runs on Host PC / Raspberry Pi with Python 3 & OpenCV
"""

import cv2
import numpy as np
import serial
import time

# Configure Serial Connection to Arduino
arduino = serial.Serial(port='COM3', baudrate=115200, timeout=0.1) # Replace with /dev/ttyUSB0 on Linux
time.sleep(2.0) # Wait for Arduino bootloader

cap = cv2.VideoCapture(0)
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)

# Initial Servo Angles
pan_angle = 90.0
tilt_angle = 90.0

# Visual Servoing Proportional Gains
Kp_pan = 0.04
Kp_tilt = 0.04

# HSV Color Filter Bounds (Tuned for Bright Orange / Tennis Ball)
LOWER_COLOR = np.array([5, 120, 120])
UPPER_COLOR = np.array([22, 255, 255])

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    frame = cv2.flip(frame, 1) # Mirror preview
    h, w, _ = frame.shape
    center_x, center_y = w // 2, h // 2

    # Color Segmentation
    hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)
    mask = cv2.inRange(hsv, LOWER_COLOR, UPPER_COLOR)
    mask = cv2.erode(mask, None, iterations=2)
    mask = cv2.dilate(mask, None, iterations=2)

    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    if contours:
        c = max(contours, key=cv2.contourArea)
        ((x, y), radius) = cv2.minEnclosingCircle(c)

        if radius > 15: # Minimum valid target radius
            cv2.circle(frame, (int(x), int(y)), int(radius), (0, 255, 0), 2)
            cv2.circle(frame, (int(x), int(y)), 5, (0, 0, 255), -1)

            # Calculate error from frame center
            error_x = x - center_x
            error_y = y - center_y

            # Visual Servoing Closed-Loop Adjustment
            pan_angle -= error_x * Kp_pan
            tilt_angle += error_y * Kp_tilt

            pan_angle = np.clip(pan_angle, 10, 170)
            tilt_angle = np.clip(tilt_angle, 20, 160)

            # Send command packet: "P<pan>T<tilt>\n"
            cmd = f"P{int(pan_angle)}T{int(tilt_angle)}\n"
            arduino.write(cmd.encode('utf-8'))

    # Draw crosshairs
    cv2.line(frame, (center_x - 15, center_y), (center_x + 15, center_y), (255, 255, 255), 1)
    cv2.line(frame, (center_x, center_y - 15), (center_x, center_y + 15), (255, 255, 255), 1)

    cv2.imshow("Tracking View", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
cv2.destroyAllWindows()
arduino.close()
```

---

## 💻 2. Arduino Firmware (C++)

```cpp
#include <Servo.h>

Servo pan_servo;
Servo tilt_servo;

const int PIN_PAN = 9;
const int PIN_TILT = 10;

void setup() {
    Serial.begin(115200);
    pan_servo.attach(PIN_PAN);
    tilt_servo.attach(PIN_TILT);

    pan_servo.write(90);
    tilt_servo.write(90);
}

void loop() {
    if (Serial.available() > 0) {
        String msg = Serial.readStringUntil('\n');
        // Parse format "P<pan>T<tilt>"
        int p_idx = msg.indexOf('P');
        int t_idx = msg.indexOf('T');

        if (p_idx != -1 && t_idx != -1) {
            int pan = msg.substring(p_idx + 1, t_idx).toInt();
            int tilt = msg.substring(t_idx + 1).toInt();

            pan = constrain(pan, 10, 170);
            tilt = constrain(tilt, 20, 160);

            pan_servo.write(pan);
            tilt_servo.write(tilt);
        }
    }
}
```
