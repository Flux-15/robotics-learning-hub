# 🛠️ Robotics Hardware & Simulation Projects Directory

<div align="center">

[ 🏠 Hub Home ](../README.md) &nbsp;•&nbsp; [ 🗺️ Full Roadmap ](../robotics-roadmap.md) &nbsp;•&nbsp; [ 📑 Hardware Pinouts Cheat Sheet ](../cheatsheets/hardware-and-pinouts.md)

</div>

---

## 📌 Overview & Scope
Theory without hardware implementation is incomplete; hardware without rigorous theory is fragile. 

Every project in this directory is a complete, production-grade **engineering blueprint** featuring:
* **System Architecture Diagrams**
* **Full Bill of Materials (BOM)** with voltage ratings, interfaces, and estimated costs
* **Detailed Electrical Wiring Schematics**
* **Complete, Runnable Source Code** with clear comments
* **Calibration & Troubleshooting Walkthroughs**

```mermaid
graph TD
    subgraph TIER1["Beginner Tier (Microcontrollers & Basic Control)"]
        P1["Line Following Robot (PID Control)"]
        P2["Obstacle Avoidance Rover (Reactive FSM)"]
        P3["Pan-Tilt Vision Turret (OpenCV Face/Color Tracking)"]
    end

    subgraph TIER2["Intermediate Tier (ROS 2, SLAM & Kinematics)"]
        P4["2D LiDAR SLAM Rover (ROS 2 Nav2 & SlamToolbox)"]
        P5["Micromouse Maze Solver (Flood-Fill Algorithm)"]
        P6["Self-Balancing Inverted Pendulum (Kalman Filter + LQR)"]
        P7["4-DoF Robotic Arm (Analytical & Numerical IK)"]
    end

    subgraph TIER3["Advanced Tier (Autonomous Systems & Embodied AI)"]
        P8["Autonomous Drone (PX4 + ROS 2 Offboard Waypoints)"]
        P9["Vision-Guided Autonomous Rover (Visual SLAM + Depth)"]
        P10["12-DoF Quadruped Robot Dog (BLDC FOC Locomotion)"]
        P11["Embodied AI Manipulator (LeRobot Diffusion Policy)"]
        P12["Multi-Robot Swarm Coordination (Decentralized Consensus)"]
    end

    TIER1 ==> TIER2 ==> TIER3

    style TIER1 fill:#1e1e2e,stroke:#a6e3a1,stroke-width:2px,color:#cdd6f4
    style TIER2 fill:#181825,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4
    style TIER3 fill:#11111b,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4
```

---

## 📂 Project Navigation Index

### 🟢 Beginner Tier
| Project | Target Concepts | Hardware Stack | Blueprint Link |
| :--- | :--- | :--- | :--- |
| **Line Following Robot** | Proportional-Derivative (PD) control, ADC array sampling, H-Bridge | Arduino Uno / Nano, 5-IR Sensor Array, L298N/TB6612 | [Build Blueprint ➔](beginner/line_following_robot.md) |
| **Obstacle Avoidance Rover** | Finite State Machine (FSM), Ultrasonic / ToF echo timing, differential drive | ESP32 / Arduino, HC-SR04, Servo scan mount | [Build Blueprint ➔](beginner/obstacle_avoidance_robot.md) |
| **Pan-Tilt Vision Turret** | Computer Vision closed-loop visual servoing, PID tracking, serial bridge | USB Camera, 2x SG90/MG996R Servos, Python OpenCV | [Build Blueprint ➔](beginner/pan_tilt_vision_turret.md) |

### 🟡 Intermediate Tier
| Project | Target Concepts | Hardware Stack | Blueprint Link |
| :--- | :--- | :--- | :--- |
| **2D LiDAR SLAM Rover** | ROS 2 Humble/Jazzy, SlamToolbox, Nav2, URDF, Odometry EKF | Raspberry Pi 4/5, RPLiDAR A1/C1, Differential Chassis | [Build Blueprint ➔](intermediate/slam_robot.md) |
| **Micromouse Maze Solver** | Graph search, Manhattan distance flood-fill, quadrature encoder odometry | STM32 / Teensy, Custom PCB, ToF sensors, DC gearmotors | [Build Blueprint ➔](intermediate/maze_solving_robot.md) |
| **Self-Balancing Robot** | Inverted pendulum dynamics, complementary/Kalman filtering, LQR | ESP32, 6-DoF MPU6050, Stepper / Encodered DC Motors | [Build Blueprint ➔](intermediate/self_balancing_robot.md) |
| **4-DoF Robotic Arm with IK** | Forward & Inverse Kinematics (PoE), geometric IK, trajectory interpolation | 4x High-Torque Servos (MG996R / Bus Servos), Python/ESP32 | [Build Blueprint ➔](intermediate/robotic_arm_ik.md) |

### 🔴 Advanced Tier
| Project | Target Concepts | Hardware Stack | Blueprint Link |
| :--- | :--- | :--- | :--- |
| **Autonomous Drone** | PX4 Autopilot, MAVROS, MAVLink, Offboard position control, VIO | Pixhawk 6C, Raspberry Pi 4 Companion, Optical Flow / Camera | [Build Blueprint ➔](advanced/autonomous_drone.md) |
| **Vision-Based Navigation** | Monocular Depth Anything v2, Visual SLAM, Obstacle avoidance | NVIDIA Jetson Orin Nano, Intel RealSense D435i, ROS 2 | [Build Blueprint ➔](advanced/vision_based_navigation_robot.md) |
| **12-DoF Quadruped Robot Dog** | Quasi-Direct Drive (QDD), FOC, Bezier curve contact gait scheduling | 12x Brushless Actuators, Teensy 4.1 / Jetson, IMU | [Build Blueprint ➔](advanced/quadruped_robot.md) |
| **Embodied AI Manipulator** | Hugging Face LeRobot, ACT / Diffusion Policy, human teleoperation | SO-ARM100 / Koch Arm, Feetech Bus Servos, PyTorch | [Build Blueprint ➔](advanced/vla_manipulator.md) |
| **Multi-Robot Swarm** | Decentralized consensus, Laplacian matrix formation, ROS 2 Zenoh | 3x TurtleBot3 or Custom Differential Rovers, Wi-Fi | [Build Blueprint ➔](advanced/multi_robot_systems.md) |

---

<div align="center">

[ ⬅️ Previous: Landmark Papers ](../research_papers/README.md) &nbsp;•&nbsp; [ 📑 Engineering Cheatsheets ➡️ ](../cheatsheets/README.md)

</div>
