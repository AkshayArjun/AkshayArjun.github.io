---
layout: post
title: "The Math Behind GlobalDrive: Splines and Predictive Control"
date: 2025-12-26 11:00:00 +0530
categories: [Theory, Control Systems]
tags: [mpc, hermite-splines, path-smoothing, c3bf]
math: true
---

This post breaks down the core mathematical and architectural decisions behind my [GlobalDrive](/posts/globaldrive-simulation/) project, specifically focusing on how to ensure smooth, optimal trajectory tracking for a differential drive robot.

---

## 1. Path Smoothing: Quintic Hermite Splines
To enable smooth motion, a path smoothing algorithm must ensure continuity in Position ($C^0$), Velocity ($C^1$), and Acceleration ($C^2$). Standard cubic splines often result in discontinuous acceleration profiles, causing jerky motion. 

To solve this, I implemented a **Quintic (5th-order) Hermite Spline Generator**. Unlike B-Splines, a Hermite spline is defined explicitly by the physical boundary conditions at the endpoints.

For a segment between two waypoints, the position $p(u)$ is defined by:
$$p(u) = c_0 + c_1 u + c_2 u^2 + c_3 u^3 + c_4 u^4 + c_5 u^5$$
where $u \in [0, 1]$ is the normalized time parameter.

To find the coefficient vector $C = [c_0, ..., c_5]^T$, we define a geometry vector $G$ containing the boundary constraints:
$$G = [p_0, v_0, a_0, p_1, v_1, a_1]^T$$

The coefficients are solved via the linear mapping $C = M \cdot G$, where $M$ is the pre-calculated **Hermite Basis Matrix**:
$$M = \begin{bmatrix}
1 & 0 & 0 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 & 0 & 0 \\
0 & 0 & 0.5 & 0 & 0 & 0 \\
-10 & -6 & -1.5 & 10 & -4 & 0.5 \\
15 & 8 & 1.5 & -15 & 7 & -1 \\
-6 & -3 & -0.5 & 6 & -3 & 0.5
\end{bmatrix}$$

**Why this works:** The Quintic formulation explicitly matches the acceleration $a_0$ of segment $i$ with $a_1$ of segment $i-1$. By explicitly calculating tangents based on the geometry of three consecutive waypoints, we can adjust the "corner sharpness" heuristic to prevent the robot from swinging wide.

---

## 2. Controller Architecture: Model Predictive Control
I chose a linearized **MPC controller** over a standard PID. PID controllers struggle with the coupled dynamics of differential drive robots. 

Our MPC solves a convex optimization problem at every time step ($10Hz$), optimizing a finite horizon ($N=40$) to minimize the following cost function:
$$J = \sum_{k=0}^{N-1} \left( \mathbf{e}_k^T \mathbf{Q} \mathbf{e}_k + \mathbf{u}_k^T \mathbf{R} \mathbf{u}_k \right)$$

Where:
* $\mathbf{e}_k = [x_{ref} - x, y_{ref} - y, \theta_{ref} - \theta]^T$ is the state error vector.
* $\mathbf{u}_k = [v, \omega]^T$ is the control input vector.
* **$\mathbf{Q}$ (State Cost Matrix):** Penalizes tracking error. High values force aggressive path tracking.
* **$\mathbf{R}$ (Input Cost Matrix):** Penalizes control effort. High values force gentle acceleration, prioritizing smoothness and energy efficiency.

---

## 3. Architectural Separation of Concerns
Instead of a monolithic script, the system is architected into three distinct modules to ensure testability:
1. **Planner (`FastSplineGenerator`):** Handles pure geometry and math. Unaware of ROS.
2. **Controller (`MPCController`):** A pure math class solving optimization problems, decoupled from hardware.
3. **Manager (`NavigationNode`):** The ROS 2 "glue" managing `/odom` callbacks and looping the controller at $10Hz$.

### Extension to Real Robots
To deploy this on a physical platform like a TurtleBot3:
* **State Estimation:** I would implement an **Extended Kalman Filter (EKF)** fusing Wheel Odometry with IMU data for a robust state estimate.
* **Computation:** For embedded hardware (e.g., Raspberry Pi), the MPC Horizon $N$ should be reduced, or the solver switched to a lighter C++ library like OSQP.

---

## 4. Safety: Control Barrier Functions (C3BF)
As an ongoing extension, I am implementing a reactive safety layer using **Control Barrier Functions**. It acts as a safety filter between the MPC and the motors.

A secondary Quadratic Program checks if the MPC's desired velocity $u_{mpc}$ will cause a collision. If unsafe, it finds the closest safe velocity $u_{safe}$ satisfying the barrier condition:
$$\dot{h}(x) \geq -\gamma h(x)$$
*Note: A common challenge currently being addressed is deadlock resolution when the MPC pushes forward but the safety function pushes back.*