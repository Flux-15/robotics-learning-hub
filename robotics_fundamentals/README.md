# 🦾 Robotics Fundamentals: Kinematics, Dynamics & Mechanics

<div align="center">

[ 🏠 Hub Home ](../README.md) &nbsp;•&nbsp; [ 🗺️ Full Roadmap ](../robotics-roadmap.md) &nbsp;•&nbsp; [ 📑 Math & Kinematics Cheat Sheet ](../cheatsheets/math-formulas-cheatsheet.md)

</div>

---

## 📌 Overview & Scope
Robotics Fundamentals constitutes the theoretical core of manipulator and mobile robot modeling. It establishes the mathematical relationship between motor joint angles/velocities and the Cartesian position, orientation, speed, and forces exerted by the robot in the 3D world.

Modern robotics has transitioned from classical Denavit-Hartenberg (D-H) parameter tables to the geometrically elegant **Product of Exponentials (PoE)** formulation rooted in Lie Groups and Screw Theory.

```mermaid
graph LR
    subgraph FORWARD["Forward Mapping"]
        J["Joint Angles (θ)"] -->|Forward Kinematics (PoE)| E["End-Effector Pose T(θ)"]
        JV["Joint Velocities (dθ/dt)"] -->|Jacobian Matrix J(θ)| EV["End-Effector Twist V"]
        ACC["Joint Accelerations"] -->|Equations of Motion| TOR["Joint Torques (τ)"]
    end

    subgraph INVERSE["Inverse Mapping"]
        E -->|Analytical / Numerical IK| J
        EV -->|Pseudo-Inverse / DLS| JV
        TOR -->|Forward Dynamics| ACC
    end

    style FORWARD fill:#1e1e2e,stroke:#89b4fa,stroke-width:2px,color:#cdd6f4
    style INVERSE fill:#181825,stroke:#f38ba8,stroke-width:2px,color:#cdd6f4
```

---

## 🔑 Core Theoretical Concepts

### 1. Forward Kinematics: D-H vs Product of Exponentials (PoE)
* **Classical Denavit-Hartenberg (D-H):** Defines 4 parameters ($a_i, \alpha_i, d_i, \theta_i$) per joint frame. However, frame assignment is ambiguous when joint axes are parallel or intersect, and small mechanical misalignments cause discontinuous parameter jumps.
* **Modern Product of Exponentials (PoE):**
  $$T(\boldsymbol{\theta}) = e^{[\mathcal{S}_1]\theta_1} e^{[\mathcal{S}_2]\theta_2} \cdots e^{[\mathcal{S}_n]\theta_n} M$$
  * $M \in SE(3)$ is the home configuration matrix of the end-effector when all joint angles $\theta_i = 0$.
  * $\mathcal{S}_i = \begin{bmatrix} \boldsymbol{\omega}_i \\ \mathbf{v}_i \end{bmatrix} \in \mathbb{R}^6$ is the spatial screw axis of joint $i$ expressed entirely in the fixed base frame $\{s\}$.
  * For a revolute joint: $\boldsymbol{\omega}_i$ is the unit axis of rotation, and $\mathbf{v}_i = -\boldsymbol{\omega}_i \times \mathbf{q}_i$ (where $\mathbf{q}_i$ is any arbitrary point along the joint axis).
  * *Advantage:* Zero frame ambiguities; smooth and continuous with respect to manufacturing variations; trivial to differentiate.

### 2. Inverse Kinematics (IK)
Given a desired Cartesian pose $T_{sd} \in SE(3)$, solve for joint angles $\boldsymbol{\theta}$ such that $T(\boldsymbol{\theta}) = T_{sd}$.
* **Analytical / Closed-Form IK:** Extremely fast (microseconds) and finds all possible solutions (e.g., 8 configurations for a 6-DoF spherical wrist arm). Possible whenever a manipulator satisfies **Pieper's Criterion** (three consecutive revolute axes intersect at a single point, like a spherical wrist).
* **Numerical IK (Iterative):** Required when closed-form solutions do not exist:
  * **Newton-Raphson:** $\boldsymbol{\theta}_{k+1} = \boldsymbol{\theta}_k + J^\dagger(\boldsymbol{\theta}_k) \, \mathbf{e}_k$, where $\mathbf{e}_k$ is the twist error.
  * **Damped Least Squares (Levenberg-Marquardt):** Prevents infinite joint velocity commands near singularities:
    $$J^* = J^T \left( J J^T + \lambda^2 I \right)^{-1}$$
    where $\lambda > 0$ is a damping factor tuned to trade off trajectory tracking precision for bounded joint speeds.

### 3. Velocity Kinematics, Jacobians & Singularities
The Geometric Jacobian $J(\boldsymbol{\theta}) \in \mathbb{R}^{6 \times n}$ relates joint velocities to the end-effector twist:
$$\mathcal{V} = \begin{bmatrix} \boldsymbol{\omega} \\ \mathbf{v} \end{bmatrix} = J(\boldsymbol{\theta}) \dot{\boldsymbol{\theta}}$$
* **Column $i$ of the Spatial Jacobian:**
  $$J_{si}(\boldsymbol{\theta}) = \left[ \text{Ad}_{e^{[\mathcal{S}_1]\theta_1} \cdots e^{[\mathcal{S}_{i-1}]\theta_{i-1}}} \right] \mathcal{S}_i$$
* **Kinematic Singularities:** Configurations where $\operatorname{rank}(J) < 6$ (or $\det(J J^T) = 0$).
  * At a singularity, the robot loses the ability to move in at least one Cartesian direction.
  * Attempting to command motion in the singular direction causes inverse kinematics to demand infinite motor speeds.
  * **Yoshikawa Manipulability Index:** $w(\boldsymbol{\theta}) = \sqrt{\det(J(\boldsymbol{\theta}) J^T(\boldsymbol{\theta}))}$. Measures distance from singularity.

### 4. Manipulator Dynamics
The fundamental equations of motion in joint space:
$$M(\boldsymbol{\theta})\ddot{\boldsymbol{\theta}} + C(\boldsymbol{\theta}, \dot{\boldsymbol{\theta}})\dot{\boldsymbol{\theta}} + \mathbf{g}(\boldsymbol{\theta}) = \boldsymbol{\tau} - J^T(\boldsymbol{\theta})\mathcal{F}_{\text{ext}}$$
* $M(\boldsymbol{\theta}) \in \mathbb{R}^{n \times n}$: Symmetric, positive-definite Mass/Inertia matrix.
* $C(\boldsymbol{\theta}, \dot{\boldsymbol{\theta}})\dot{\boldsymbol{\theta}} \in \mathbb{R}^n$: Coriolis and centrifugal forces.
* $\mathbf{g}(\boldsymbol{\theta}) \in \mathbb{R}^n$: Gravitational torque vector (used for gravity compensation control).
* $J^T \mathcal{F}_{\text{ext}}$: External forces/moments mapped directly to joint torques via the Jacobian transpose.

---

## 🛠️ Industry Standard Computational Tools

| Tool | Language | Key Feature |
| :--- | :--- | :--- |
| **[Modern Robotics Code Library](https://github.com/NxRLab/ModernRobotics)** | Python / C++ | Canonical implementation of PoE kinematics, spatial Jacobians, and dynamics |
| **[Pinocchio](https://github.com/stack-of-tasks/pinocchio)** | C++ / Python | State-of-the-art rigid body dynamics library using Spatial Vector Algebra |
| **[Robotics Toolbox for Python](https://github.com/petercorke/robotics-toolbox-python)** | Python | Peter Corke's comprehensive kinematics, trajectory, and arm visualization tool |
| **[KDL (Orocos Kinematics and Dynamics)](https://github.com/orocos/orocos_kinematics_dynamics)** | C++ | Traditional kinematics solver integrated directly with ROS and URDF |

---

## 📚 Curated Learning Media

### 🎓 Premier Courses & Lectures
- [Modern Robotics: Mechanics, Planning, and Control (Prof. Kevin Lynch, Northwestern)](https://www.youtube.com/playlist?list=PLggLP4f-rq02vX0OQQ5vrCxbJrzamYDfx) — The definitive modern video course on screw theory and manipulator dynamics.
- [Stanford CS223A: Introduction to Robotics (Prof. Oussama Khatib)](https://www.youtube.com/playlist?list=PL656D8C9E41DE3171) — Foundational kinematics and operational space control.
- [Mobile Robotics (Prof. Cyrill Stachniss, University of Bonn / Freiburg)](https://www.youtube.com/playlist?list=PLgnQpQtFTOGRsqKaaHwFq0A0BwS8j3oA5) — Wheeled mobile robot kinematics, unicycle model, and odometry.

### 📖 Essential Textbooks
- *Modern Robotics: Mechanics, Planning, and Control* by Kevin M. Lynch and Frank C. Park (Free online PDF).
- *Introduction to Robotics: Mechanics and Control* by John J. Craig.
- *Springer Handbook of Robotics* by Bruno Siciliano and Oussama Khatib.
- *Robotics, Vision and Control* by Peter Corke.

---

<div align="center">

[ ⬅️ Previous: Mechanical Design ](../mechanical_design/README.md) &nbsp;•&nbsp; [ Next: Control Systems ➡️ ](../control_systems/README.md)

</div>
