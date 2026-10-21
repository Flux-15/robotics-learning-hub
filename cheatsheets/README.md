# 📑 Robotics Engineering Cheat Sheets & Rapid Reference Matrix

<div align="center">

[ 🏠 Hub Home ](../README.md) &nbsp;•&nbsp; [ 🗺️ Full Roadmap ](../robotics-roadmap.md) &nbsp;•&nbsp; [ 🛠️ Hardware Projects ](../robotics_projects/resources.md)

</div>

---

## 📌 Overview & Scope
During laboratory bringup, debugging at the test bench, or coding late at night, you don't want to re-read a 600-page textbook just to recall the syntax for a ROS 2 lifecycle transition, the Rodrigues rotation matrix formula, an active CAN-FD bus termination check, or a `udev` hardware rule.

This directory houses **field-tested, condensed cheat sheets** engineered for immediate reference.

```mermaid
graph TD
    CHEATS["Engineering Cheat Sheets Directory"]

    C1["🤖 ROS 2 & Middleware Cheatsheet<br/>CLI commands, QoS, Colcon, TF2, Launch syntax"]
    C2["📐 Mathematics & Kinematics Cheatsheet<br/>SO(3), SE(3), Quaternions, PoE, Jacobians, LQR, EKF"]
    C3["💻 Linux, Git & Systems Cheatsheet<br/>udev rules, systemd units, PREEMPT_RT, CMake, Docker"]
    C4["⚡ Electronics, Hardware & Pinouts Cheatsheet<br/>CAN bus, SPI/I2C/UART, LiPo thresholds, AWG gauges"]

    CHEATS --> C1
    CHEATS --> C2
    CHEATS --> C3
    CHEATS --> C4

    style CHEATS fill:#1e1e2e,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4
    style C1 fill:#181825,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4
    style C2 fill:#11111b,stroke:#f9e2af,stroke-width:2px,color:#cdd6f4
    style C3 fill:#181825,stroke:#a6e3a1,stroke-width:2px,color:#cdd6f4
    style C4 fill:#1e1e2e,stroke:#cba6f7,stroke-width:2px,color:#cdd6f4
```

---

## 📑 Quick Navigation

| Cheat Sheet | Primary Contents | Direct Link |
| :--- | :--- | :--- |
| **🤖 ROS 2 CLI & Nav2** | Topic/Service/Action commands, Lifecycle nodes, TF2 debugging, QoS syntax, Colcon flags | [Open Cheat Sheet ➔](ros2-cheatsheet.md) |
| **📐 Mathematics & Kinematics** | Rotation matrices, Quaternion operations, PoE formula, Spatial Jacobians, LQR Riccati, Kalman Filter | [Open Cheat Sheet ➔](math-formulas-cheatsheet.md) |
| **💻 Linux, Git & Tooling** | Persistent `/dev` symlinks (udev), systemd startup services, real-time priority, CMake targets, Git LFS | [Open Cheat Sheet ➔](linux-and-git-cheatsheet.md) |
| **⚡ Hardware, Bus & Pinouts** | CAN bus termination, SPI vs I2C vs UART wiring, LiPo cell voltage limits, AWG wire sizing, Motor torque | [Open Cheat Sheet ➔](hardware-and-pinouts.md) |

---

<div align="center">

[ ⬅️ Return to Hub Home ](../README.md)

</div>
