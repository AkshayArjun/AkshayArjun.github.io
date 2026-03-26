---
layout: post
title: "AgileDrone: RL-Based Autonomous UAV Control"
date: 2025-10-09 12:00:00 +0530
categories: [Projects, Robotics]
tags: [reinforcement-learning, drones, genesis, px4, ros2]
image:
  path: /assets/img/projects/agileDrone/x500_genesis.png
  alt: AgileDrone Tarot 650 in Genesis Simulation
---

**AgileDrone** is an end-to-end Reinforcement Learning framework designed to achieve high-agility autonomous flight. By training a neural network "pilot" in physics-accurate simulation environments, the system learns to map raw sensor observations directly to motor thrusts — bridging the gap between simulation and real-world hardware deployment on a Tarot 650 quadcopter.

You can view the full source code and technical implementation on my [GitHub Repository](https://github.com/AkshayArjun/agileDrone).

### The Challenge

Standard GNC (Guidance, Navigation, and Control) stacks for drones often break down at high speeds and aggressive maneuvers, where non-linear aerodynamics become the dominant factor. Tuning PID gains for every possible flight regime is exhaustive and brittle — a controller tuned for hover may become unstable at high velocity, and vice versa.

The goal of this project was to create a **unified policy** that handles everything from aggressive recovery from extreme orientations to precise hovering, without manual gain scheduling or mode-switching logic.

### The Approach

I developed a multi-stage pipeline that takes an agent from zero-knowledge initialization to hardware-ready deployment:

**1. High-Fidelity Simulation**

Using the [Genesis](https://genesis-world.readthedocs.io) and [Isaac Lab](https://isaac-sim.github.io/IsaacLab/) physics engines, I trained a **PPO (Proximal Policy Optimization)** agent. The agent observes a **17-dimensional state vector** — encoding relative position, orientation quaternion, and linear/angular velocities — and outputs normalized motor thrust commands at a **100 Hz control loop**.

**2. Reward Function Design**

To shape the agent's behavior, I engineered a reward system that balances mission objectives with hardware safety constraints:

- **Target Reward:** Shaped by the change in squared distance to the goal $(\Delta \|p - p_{\text{goal}}\|^2)$, forcing the agent to seek the target aggressively rather than passively drifting toward it.
- **Yaw & Angular Penalties:** Encourages heading stability and penalizes high-frequency oscillations that would be unphysical on real hardware.
- **Action Smoothness:** Penalizes the derivative of motor commands $(\|a_t - a_{t-1}\|^2)$ to prevent jitter, reduce mechanical wear, and improve battery efficiency.

**3. Sim2Real & Deployment**

Trained PyTorch policies were exported to **ONNX** for low-latency inference on a companion computer. The companion computer interfaces with a **PX4 Autopilot** via **ROS 2**, injecting high-level setpoints into the flight controller's offboard control mode.

### Results

The Tarot 650 agent successfully demonstrated **stable hovering** and the ability to **recover from extreme initial orientations** (tilts exceeding 60°), validating that a single unified policy can handle the full operating envelope without mode switching.

**Genesis Simulation Viewer**

![AgileDrone in Genesis](/assets/img/projects/agileDrone/x500_genesis.png)
*The Tarot 650 quadcopter maintaining stability in the Genesis interactive viewer.*

**[Watch the Agile Flight Demo Video Here](/assets/img/projects/agileDrone/agileDrone.webm)**

### Technical Breakdown

| Component | Details |
|---|---|
| **Frame** | Tarot 650 Assembly (Custom URDF, Convex Decomposition) |
| **Physics Engines** | Genesis & NVIDIA Isaac Lab |
| **RL Algorithm** | PPO (Proximal Policy Optimization) |
| **Observation Space** | 17-dim (position, orientation, lin/ang velocity) |
| **Control Frequency** | 100 Hz |
| **Software Stack** | ROS 2 Humble, PX4 Autopilot, ONNX Runtime |
| **Deployment Target** | Companion Computer → PX4 Offboard Mode |