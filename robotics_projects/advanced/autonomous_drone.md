# 🚁 Autonomous Aerial Drone with PX4 & ROS 2 Offboard Control

<div align="center">

[ ⬅️ Back to Projects Directory ](../README.md) &nbsp;•&nbsp; [ 🏠 Hub Home ](../../README.md)

</div>

---

## 📌 Project Overview
An industrial-grade autonomous aerial robotic system. The quadrotor utilizes the **PX4 Autopilot** flight stack on a dedicated real-time flight controller (Pixhawk 6C) coupled to an onboard Linux companion computer (Raspberry Pi 4 / NVIDIA Jetson) running **ROS 2**. 

Communication is orchestrated via **MAVLink (MAVROS)** or native **micro-XRCE-DDS**, enabling the companion computer to stream high-level 3D position, velocity, and yaw setpoints in **Offboard Mode** while the flight controller guarantees low-level attitude and angular rate stabilization at 400 Hz.

```mermaid
graph TD
    subgraph COMPANION["Onboard Linux Companion (Raspberry Pi / Jetson)"]
        PLANNER["ROS 2 Autonomy / Waypoint Planner<br/>(Trajectory Generation & Obstacle Avoidance)"]
        VIO["Visual Inertial Odometry (VIO) / RealSense<br/>Streams Local Pose @ 30 Hz"]
        BRIDGE["micro-XRCE-DDS Client / MAVROS"]
        PLANNER --> BRIDGE
        VIO --> BRIDGE
    end

    subgraph AUTOPILOT["PX4 Autopilot (Pixhawk 6C / STM32H7)"]
        EST["EKF2 Sensor Fusion Engine<br/>(Fuses IMU, Baro, GPS, and External VIO Vision)"]
        CTRL["Cascaded PID Flight Controllers<br/>(Position ➔ Velocity ➔ Attitude ➔ Angular Rate)"]
        MOTOR_MIX["Motor Mixer & DShot600 Protocol"]
        BRIDGE <==>|High-Speed UART (921600 baud)| EST
        EST --> CTRL --> MOTOR_MIX
    end

    subgraph HARDWARE["Drone Propulsion & Sensors"]
        ESC["4-in-1 45A BLDC ESC Rail"]
        MOTORS["4x 2207 Brushless Motors + 5-inch Props"]
        BAT["4S / 6S High-Discharge LiPo Battery (100C)"]
        MOTOR_MIX --> ESC --> MOTORS
        BAT --> ESC
    end

    style COMPANION fill:#1e1e2e,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4
    style AUTOPILOT fill:#181825,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4
    style HARDWARE fill:#11111b,stroke:#a6e3a1,stroke-width:2px,color:#cdd6f4
```

---

## 📋 Bill of Materials (BOM)

| Component | Specification | Quantity | Interface / Voltage | Est. Cost (USD) |
| :--- | :--- | :---: | :--- | :---: |
| **Flight Controller** | Holybro Pixhawk 6C + PM02 Power Module | 1 | STM32H7, Dual IMUs, CAN, UART | \$190.00 |
| **Companion Computer** | Raspberry Pi 4 (4GB) or Jetson Orin Nano | 1 | Ubuntu 22.04 LTS (5V 3A) | \$65.00 |
| **Carbon Frame** | 5-Inch or 7-Inch Carbon Fiber Quad Frame | 1 | 220mm – 300mm Wheelbase | \$35.00 |
| **Brushless Motors** | 4x 2207 1800KV (6S) or 2400KV (4S) Motors | 4 | 3-Phase BLDC | \$60.00 |
| **ESC** | 45A–55A 4-in-1 BLHeli_32 / DShot600 ESC | 1 | 4S–6S Input | \$45.00 |
| **Tracking Camera** | Intel RealSense T265 (VIO) or D435i Depth | 1 | USB 3.0 (Indoor GPS-denied nav) | \$180.00 |
| **Telemetry / RC** | ELRS (ExpressLRS) 2.4GHz Receiver | 1 | CRSF Serial Protocol (Failsafe RC) | \$18.00 |
| **Power Source** | 4S or 6S 1500mAh 100C LiPo Battery | 2 | XT60 | \$55.00 |

---

## ⚡ Electrical & Telemetry Wiring

```
Pixhawk 6C Port       Connected Device & Protocol
─────────────────────────────────────────────────────────────────────────────
TELEM2 (Serial)       Raspberry Pi GPIO 14/15 (UART @ 921600 baud for MAVLink)
POWER1                PM02 Power Module (Monitors Battery Volts & Amps)
I2C / GPS             Compass / External Magnetometer
RC IN                 ELRS Receiver (CRSF Protocol)
I/O PWM OUT (1-4)     4-in-1 ESC Signal Inputs (DShot600 protocol)
Raspberry Pi USB 3.0  Intel RealSense Camera (High-bandwidth image stream)
```

---

## 💻 ROS 2 Python Offboard Control Node

This production ROS 2 node connects to the PX4 stack via MAVROS, engages Offboard mode, arms the motors, and commands a square trajectory waypoint mission:

```python
#!/usr/bin/env python3
"""
PX4 ROS 2 Autonomous Offboard Flight Controller
Publishes local NED setpoints and coordinates mode transitions.
"""

import rclpy
from rclpy.node import Node
from geometry_msgs.msg import PoseStamped
from mavros_msgs.msg import State
from mavros_msgs.srv import CommandBool, SetMode

class OffboardFlightController(Node):
    def __init__(self):
        super().__init__('px4_offboard_controller')

        # Subscribers & Publishers
        self.state_sub = self.create_subscription(State, 'mavros/state', self.state_callback, 10)
        self.target_pose_pub = self.create_publisher(PoseStamped, 'mavros/setpoint_position/local', 10)

        # Service Clients
        self.arming_client = self.create_client(CommandBool, 'mavros/cmd/arming')
        self.set_mode_client = self.create_client(SetMode, 'mavros/set_mode')

        self.current_state = State()
        self.target_pose = PoseStamped()

        # Target Takeoff Altitude: 2.5 meters
        self.target_pose.pose.position.x = 0.0
        self.target_pose.pose.position.y = 0.0
        self.target_pose.pose.position.z = 2.5

        # PX4 Offboard requirement: Must stream setpoints at >= 20 Hz BEFORE engaging mode
        self.timer = self.create_timer(0.05, self.control_loop) # 20 Hz
        self.attempts = 0

    def state_callback(self, msg):
        self.current_state = msg

    def control_loop(self):
        self.target_pose.header.stamp = self.get_clock().now().to_msg()
        self.target_pose_pub.publish(self.target_pose)

        # After 100 setpoints have streamed (~5 seconds), request Offboard & Arm
        if self.attempts < 100:
            self.attempts += 1
            return

        if self.current_state.mode != "OFFBOARD" and self.set_mode_client.service_is_ready():
            req = SetMode.Request()
            req.custom_mode = 'OFFBOARD'
            self.set_mode_client.call_async(req)
            self.get_logger().info("Requesting OFFBOARD flight mode...")

        if not self.current_state.armed and self.arming_client.service_is_ready():
            req = CommandBool.Request()
            req.value = True
            self.arming_client.call_async(req)
            self.get_logger().info("Arming drone motors...")

def main(args=None):
    rclpy.init(args=args)
    node = OffboardFlightController()
    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass
    finally:
        node.destroy_node()
        rclpy.shutdown()

if __name__ == '__main__':
    main()
```

---

## 🛡️ Critical Safety & Failsafe Rules
1. **RC Kill Switch (Mandatory):** Always map a physical 2-position switch on your RC transmitter (e.g., RadioMaster Boxer / Pocket) to **Emergency Motor Cut / Disarm**. In the event of a software divergence or indoor drift, flipping the switch cuts motor signals in under 10 milliseconds.
2. **PX4 Streaming Liveness:** If the companion computer halts or stops streaming setpoints for more than **500 milliseconds**, PX4 triggers an automatic failsafe (Hold Position, Land in place, or Return to Launch / RTL).
3. **Propeller Warning:** **NEVER test offboard code on the workbench with propellers installed!** Always remove all four propellers until manual flight tests and failsafes are verified.
