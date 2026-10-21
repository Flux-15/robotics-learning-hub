# ⚡ Electronics, Hardware & Pinouts Cheat Sheet

<div align="center">

[ ⬅️ Back to Cheatsheets ](README.md) &nbsp;•&nbsp; [ 🏠 Hub Home ](../README.md) &nbsp;•&nbsp; [ ⚡ Electronics Module ](../electronics/README.md)

</div>

---

## 🔋 LiPo & LiFePO4 Battery Voltage Thresholds

| State | LiPo (Per Cell) | 3S LiPo (11.1V) | 4S LiPo (14.8V) | 6S LiPo (22.2V) | LiFePO4 (Per Cell) |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Maximum Charged** | **4.20V** | 12.60V | 16.80V | 25.20V | 3.65V |
| **Nominal Voltage** | **3.70V** | 11.10V | 14.80V | 22.20V | 3.20V |
| **Storage Voltage** | **3.85V** | 11.55V | 15.40V | 23.10V | 3.30V |
| **Critical Cutoff** | **3.30V** | 9.90V | 13.20V | 19.80V | 2.50V |

> [!CAUTION]
> Discharging a LiPo below 3.0V per cell causes irreversible copper shunts and permanent internal resistance degradation! Always program your microcontroller or BMS to sound an alarm at **3.50V / cell** under load.

---

## 🧵 American Wire Gauge (AWG) Power Sizing

Selecting wire that is too thin will melt insulation and drop voltage during motor acceleration surges:

| Wire Gauge | Diameter (mm) | Max Continuous Current (Amps) | Typical Robotics Application |
| :---: | :---: | :---: | :--- |
| **10 AWG** | 2.59 mm | **55 A – 70 A** | Main battery lead for heavy mobile bases / humanoids |
| **12 AWG** | 2.05 mm | **40 A – 50 A** | High-power BLDC motor phase wires, 6S ESC power leads |
| **14 AWG** | 1.63 mm | **25 A – 35 A** | 4S quadcopter ESCs, high-torque DC motor drivers |
| **18 AWG** | 1.02 mm | **10 A – 14 A** | Step-down buck converters, 12V SBC power input |
| **22 AWG** | 0.64 mm | **3 A – 5 A** | High-power servo leads, motor driver logic rails |
| **28 AWG** | 0.32 mm | **< 1 A** | I2C, SPI, UART signal wiring, encoder differential pairs |

---

## 📡 Sensor & Communication Protocol Pinouts

### 1. CAN Bus (Controller Area Network)
* `CAN_H`: High-level differential signal (reaches 3.5V during dominant bit).
* `CAN_L`: Low-level differential signal (drops to 1.5V during dominant bit).
* **Termination Rule:** Measure resistance across `CAN_H` and `CAN_L` with power off. An unpowered, correctly terminated bus must measure **$60.0 \ \Omega$** (two $120\ \Omega$ resistors in parallel at opposite bus ends).

### 2. SPI (Serial Peripheral Interface)
```
Master (MCU)                        Slave (IMU / Encoder / Flash)
MOSI (Master Out Slave In)  ────>   SDI (Serial Data In)
MISO (Master In Slave Out)  <────   SDO (Serial Data Out)
SCK (Serial Clock)          ────>   SCLK (Clock)
CS / SS (Chip Select)       ────>   CS (Active LOW)
```

### 3. I2C (Inter-Integrated Circuit)
* `SDA` (Serial Data) & `SCL` (Serial Clock).
* Open-drain bus: requires external **$2.2\text{k}\Omega - 4.7\text{k}\Omega$ pull-up resistors** to $3.3V$ on both lines if internal MCU pull-ups are weak.

---

## ⚙️ Motor Sizing Quick Formulas

1. **Torque Required for Acceleration:**
   $$\tau = \tau_{\text{static}} + I_{\text{total}} \cdot \alpha = m g r + (I_m \cdot N^2 + I_{\text{load}}) \frac{\omega_{\text{target}}}{t_{\text{accel}}}$$
2. **Mechanical Power:**
   $$P_{\text{mech}} \text{ (Watts)} = \tau \text{ (N}\cdot\text{m)} \times \omega \text{ (rad/s)}$$
   $$\text{RPM to rad/s: } \omega = \text{RPM} \times \frac{2\pi}{60} \approx \text{RPM} \times 0.1047$$
3. **KV Rating for BLDC Motors:**
   $$\text{No-Load RPM} = \text{KV} \times V_{\text{battery}}$$
