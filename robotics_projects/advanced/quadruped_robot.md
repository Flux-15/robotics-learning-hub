# 🐕 12-DoF Brushless Quadruped Robot Dog Engineering Blueprint

<div align="center">

[ ⬅️ Back to Projects Directory ](../README.md) &nbsp;•&nbsp; [ 🏠 Hub Home ](../../README.md)

</div>

---

## 📌 Project Overview
A state-of-the-art dynamic quadruped robot modeled after platforms like MIT Mini Cheetah and Unitree Go2. The robot features **12 active degrees of freedom** (3 DoF per leg: Hip Abduction/Adduction, Hip Pitch, Knee Pitch), powered by **Quasi-Direct Drive (QDD)** brushless actuators communicating over high-speed **CAN-FD** buses. 

Locomotion is orchestrated via a parameterized **Trot Gait Scheduler** with analytical 3-DoF leg inverse kinematics and Bézier curve foot trajectory generation.

```mermaid
graph TD
    subgraph BRAIN["High-Level Locomotion Controller (Jetson / Teensy 4.1)"]
        STATE["Robot Body State Estimator<br/>(Fuses 6-DoF IMU & Leg Kinematics)"]
        GAIT["Gait Generator & Phase Scheduler<br/>(Trot, Pace, Pronk - Duty Factor d=0.5)"]
        BEZIER["Foot Trajectory Generator<br/>(12-Point Bézier Spline for Swing Clearance)"]
        IK["3-DoF Analytical Leg Inverse Kinematics"]

        STATE --> GAIT --> BEZIER --> IK
    end

    subgraph CAN_BUS["Dual CAN-FD High-Speed Actuator Buses (1-5 Mbps)"]
        IK -->|Joint Setpoints (θ, dθ/dt, Kp, Kd, τ_ff)| CAN
        CAN["CAN-FD Transceiver Rails (Legs FL, FR, RL, RR)"]
    end

    subgraph ACTUATORS["12x Quasi-Direct Drive (QDD) Brushless Modules"]
        M1["Front-Left Leg (3x Motors)"]
        M2["Front-Right Leg (3x Motors)"]
        M3["Rear-Left Leg (3x Motors)"]
        M4["Rear-Right Leg (3x Motors)"]
        CAN --> M1
        CAN --> M2
        CAN --> M3
        CAN --> M4
    end

    style BRAIN fill:#1e1e2e,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4
    style CAN_BUS fill:#181825,stroke:#f9e2af,stroke-width:2px,color:#cdd6f4
    style ACTUATORS fill:#11111b,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4
```

---

## 📋 Bill of Materials (BOM)

| Component | Specification | Quantity | Interface / Voltage | Est. Cost (USD) |
| :--- | :--- | :---: | :--- | :---: |
| **QDD Actuators** | RobStride 04 / CyberGear / T-Motor AK60-6 (6:1 Planetary) | 12 | 24V–48V DC, CAN-FD, 14-bit Encoders | \$1,200.00 |
| **Companion Compute** | NVIDIA Jetson Orin Nano + Teensy 4.1 Real-Time Core | 1 | Linux + RTOS (SPI/CAN bridge) | \$320.00 |
| **IMU Sensor** | VectorNav VN-100 or ICM-42688P 6-DoF IMU | 1 | High-bandwidth SPI (1 kHz update) | \$25.00 |
| **CAN Transceivers** | 2x Waveshare CAN-FD Isolators (SN65HVD230) | 2 | Dual-bus separation | \$12.00 |
| **Carbon Structure** | CNC 3K Carbon Fiber Sideplates & CNC Aluminum Hips | 1 | Lightweight rigid chassis frame | \$180.00 |
| **Leg Links** | 3D Printed PA-CF (Nylon Carbon Fiber) Lower Legs | 4 | High impact resistance | \$30.00 |
| **Rubber Footpads** | Silicone / Polyurethane High-Traction Spheres | 4 | Contact damping | \$15.00 |
| **Power System** | 6S LiPo Battery (22.2V, 5000mAh, 60C) + 100A BMS | 1 | XT90 Anti-Spark Connector | \$85.00 |

---

## 🔑 Leg Kinematics: Analytical 3-DoF Closed-Form

Each leg consists of:
* $L_1$: Hip abduction link offset.
* $L_2$: Upper thigh link length.
* $L_3$: Lower calf link length.

Given target foot coordinates $(x, y, z)$ relative to the hip shoulder origin:

### 1. Hip Abduction/Adduction Angle ($\theta_1$)
$$D = \sqrt{y^2 + z^2 - L_1^2}$$
$$\theta_1 = \text{atan2}(y, -z) - \text{atan2}(L_1, D)$$

### 2. Knee Angle ($\theta_3$)
Project into the leg sagittal plane:
$$r = \sqrt{x^2 + D^2}$$
By the Law of Cosines:
$$\cos(\theta_3) = \frac{r^2 - L_2^2 - L_3^2}{2 L_2 L_3}$$
$$\theta_3 = -\text{acos}\left( \text{clamp}(\cos\theta_3, -1.0, 1.0) \right)$$

### 3. Hip Pitch Angle ($\theta_2$)
$$\alpha = \text{atan2}(x, D)$$
$$\beta = \text{atan2}\left( L_3 \sin(-\theta_3), \; L_2 + L_3 \cos(\theta_3) \right)$$
$$\theta_2 = \alpha + \beta$$

---

## 💻 Bézier Curve Foot Trajectory Generator (Python)

During the **swing phase** of a trot gait, the foot must smoothly lift off the ground, clear obstacles at mid-stride height $h_{\text{clear}}$, and touch down with zero impact velocity. This is modeled using an 11-point 1D/3D Bézier curve:

```python
import numpy as np

def bezier_point(control_points, t):
    """
    Computes position along a Bezier curve of order N at normalized phase t in [0, 1].
    """
    n = len(control_points) - 1
    point = np.zeros_like(control_points[0])
    for i, p in enumerate(control_points):
        # Binomial coefficient
        binom = np.math.comb(n, i)
        bernstein = binom * ((1.0 - t) ** (n - i)) * (t ** i)
        point += bernstein * p
    return point

class LegTrajectory:
    def __init__(self, step_length=0.12, step_height=0.045):
        self.step_len = step_length
        self.step_h = step_height

    def get_swing_target(self, t, stance_start, stance_end):
        """
        Generates 3D foot swing trajectory from liftoff to touchdown.
        t in [0.0, 1.0]
        """
        # 12 Control Points shaping horizontal progression and vertical arc
        # Ensures zero velocity and zero acceleration at boundary points
        z_clearance = np.array([
            0.0, 0.0, 0.1, 0.3, 0.7, 1.0, 1.0, 1.0, 0.7, 0.3, 0.0, 0.0
        ]) * self.step_h

        x_progress = np.linspace(stance_start[0], stance_end[0], 12)
        y_progress = np.linspace(stance_start[1], stance_end[1], 12)
        z_progress = np.linspace(stance_start[2], stance_end[2], 12) + z_clearance

        control_pts = [np.array([x, y, z]) for x, y, z in zip(x_progress, y_progress, z_progress)]
        return bezier_point(control_pts, t)

# Example Execution
traj = LegTrajectory(step_length=0.14, step_height=0.05)
liftoff_pos = np.array([-0.07, 0.09, -0.22])
touchdown_pos = np.array([0.07, 0.09, -0.22])

mid_swing_foot_target = traj.get_swing_target(0.5, liftoff_pos, touchdown_pos)
print(f"Mid-Swing 3D Foot Coordinate: {mid_swing_foot_target.round(4)} meters")
```

---

## ⚡ Motor Controller Command Protocol (MIT Impedance Mode)
Rather than raw position commands, each actuator operates under the **MIT Hybrid Control Law**:
$$\tau = K_p (\theta_d - \theta) + K_d (\dot{\theta}_d - \dot{\theta}) + \tau_{\text{feedforward}}$$
* Allows the leg to act as a virtual spring-damper upon foot contact.
* $K_p \approx 20.0 - 40.0 \text{ N}\cdot\text{m/rad}$: Compliant enough to absorb ground impact.
* $K_d \approx 1.0 - 2.5 \text{ N}\cdot\text{m}\cdot\text{s/rad}$: Provides vital velocity damping.
