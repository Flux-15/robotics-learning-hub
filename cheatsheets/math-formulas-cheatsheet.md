# 📐 Robotics Mathematics & Kinematics Formula Sheet

<div align="center">

[ ⬅️ Back to Cheatsheets ](README.md) &nbsp;•&nbsp; [ 🏠 Hub Home ](../README.md) &nbsp;•&nbsp; [ 📐 Mathematics Module ](../mathematics/README.md)

</div>

---

## 🔄 3D Rotations & $SO(3)$

### 1. Rodrigues' Rotation Formula
Converts an axis-angle representation $(\hat{\boldsymbol{\omega}}, \theta)$ with $\|\hat{\boldsymbol{\omega}}\| = 1$ into an $SO(3)$ matrix:
$$R = \exp([\hat{\boldsymbol{\omega}}]_\times \theta) = I + \sin\theta [\hat{\boldsymbol{\omega}}]_\times + (1 - \cos\theta) [\hat{\boldsymbol{\omega}}]_\times^2$$
where the skew-symmetric matrix $[\hat{\boldsymbol{\omega}}]_\times$ is:
$$[\hat{\boldsymbol{\omega}}]_\times = \begin{bmatrix} 0 & -\omega_z & \omega_y \\ \omega_z & 0 & -\omega_x \\ -\omega_y & \omega_x & 0 \end{bmatrix}$$

### 2. Unit Quaternions
Represented as $q = (w, x, y, z) = (\cos\frac{\theta}{2}, \hat{\mathbf{u}}\sin\frac{\theta}{2})$ with $\|q\| = 1$:
* **Quaternion Inverse / Conjugate:**
  $$q^{-1} = q^* = (w, -x, -y, -z)$$
* **Quaternion Multiplication (Hamilton Product):**
  $$q_1 \otimes q_2 = \begin{bmatrix} w_1 w_2 - \mathbf{v}_1 \cdot \mathbf{v}_2 \\ w_1 \mathbf{v}_2 + w_2 \mathbf{v}_1 + \mathbf{v}_1 \times \mathbf{v}_2 \end{bmatrix}$$
* **Rotating a 3D Vector $\mathbf{p}$:**
  $$\mathbf{p}_{\text{rotated}} = q \otimes (0, \mathbf{p}) \otimes q^*$$

---

## 🦾 Rigid Body Motions & $SE(3)$

### 1. Homogeneous Transformation Matrix
$$T = \begin{bmatrix} R & \mathbf{p} \\ \mathbf{0}_{1 \times 3} & 1 \end{bmatrix} \in SE(3)$$
* **Analytical Matrix Inversion (Fast & Exact):**
  $$T^{-1} = \begin{bmatrix} R^T & -R^T \mathbf{p} \\ \mathbf{0}_{1 \times 3} & 1 \end{bmatrix}$$

### 2. Modern Product of Exponentials (PoE) Forward Kinematics
$$T(\boldsymbol{\theta}) = e^{[\mathcal{S}_1]\theta_1} e^{[\mathcal{S}_2]\theta_2} \cdots e^{[\mathcal{S}_n]\theta_n} M$$
* $\mathcal{S}_i = \begin{bmatrix} \hat{\boldsymbol{\omega}}_i \\ \mathbf{v}_i \end{bmatrix} \in \mathbb{R}^6$: Screw axis in base frame.
* Revolute joint: $\hat{\boldsymbol{\omega}}_i = \text{axis unit vector}$, $\mathbf{v}_i = -\hat{\boldsymbol{\omega}}_i \times \mathbf{q}_i$.
* Prismatic joint: $\hat{\boldsymbol{\omega}}_i = \mathbf{0}$, $\mathbf{v}_i = \text{translation unit vector}$.

### 3. Spatial Velocity & Geometric Jacobian
$$\mathcal{V}_s = \begin{bmatrix} \boldsymbol{\omega}_s \\ \mathbf{v}_s \end{bmatrix} = J_s(\boldsymbol{\theta}) \dot{\boldsymbol{\theta}}$$
* **Column $i$ of the Spatial Jacobian:**
  $$J_{si}(\boldsymbol{\theta}) = \text{Ad}_{e^{[\mathcal{S}_1]\theta_1} \cdots e^{[\mathcal{S}_{i-1}]\theta_{i-1}}} \mathcal{S}_i$$
* **Yoshikawa Manipulability Measure:**
  $$w(\boldsymbol{\theta}) = \sqrt{\det(J(\boldsymbol{\theta}) J^T(\boldsymbol{\theta}))}$$
* **Damped Least Squares (DLS) Inverse Kinematics:**
  $$\Delta \boldsymbol{\theta} = J^T \left( J J^T + \lambda^2 I \right)^{-1} \mathbf{e}$$

---

## 🎛️ Control Systems & Optimization

### 1. Continuous Linear Quadratic Regulator (LQR)
For linear system $\dot{\mathbf{x}} = A\mathbf{x} + B\mathbf{u}$:
* **Optimal Gain:** $K = R^{-1} B^T P$
* **Algebraic Riccati Equation (ARE):**
  $$A^T P + P A - P B R^{-1} B^T P + Q = 0$$
* **Control Law:** $\mathbf{u} = -K \mathbf{x}$

### 2. Discrete Kalman Filter (Optimal State Estimator)
$$\text{System: } \mathbf{x}_k = A \mathbf{x}_{k-1} + B \mathbf{u}_k + \mathbf{w}_k, \quad \mathbf{z}_k = C \mathbf{x}_k + \mathbf{v}_k$$
* **Prediction:**
  $$\hat{\mathbf{x}}_{k|k-1} = A \hat{\mathbf{x}}_{k-1|k-1} + B \mathbf{u}_k$$
  $$P_{k|k-1} = A P_{k-1|k-1} A^T + Q_k$$
* **Kalman Gain:**
  $$K_k = P_{k|k-1} C^T \left( C P_{k|k-1} C^T + R_k \right)^{-1}$$
* **Update / Correction:**
  $$\hat{\mathbf{x}}_{k|k} = \hat{\mathbf{x}}_{k|k-1} + K_k \left( \mathbf{z}_k - C \hat{\mathbf{x}}_{k|k-1} \right)$$
  $$P_{k|k} = (I - K_k C) P_{k|k-1}$$
