---
layout: post
title: "GlobalDrive: Autonomous Differential Drive Simulation"
date: 2025-12-26 10:00:00 +0530
categories: [Projects, ROS 2]
tags: [gazebo, trajectory-planning, mpc, turtlebot3]
image:
    path: /assets/img/projects/globalDrive/straight_line.png
    alt: GlobbalDrive LPV-MPC Simulation
---

**GlobalDrive** is a complete ROS 2 navigation stack that simulates the motion of a differential drive robot—handling everything from global path planning to local trajectory tracking and control within a Gazebo Harmonic environment.

You can view the full source code and installation instructions on my [GitHub Repository](https://github.com/AkshayArjun/globalDrive).

### The Challenge
Navigating a differential drive robot (like a TurtleBot3) from point A to point B sounds simple, but doing it *smoothly* in real-time is highly complex. Standard linear pathing causes jerky movements and massive torque spikes in the motors. Furthermore, standard reactive controllers (like PID) struggle with the coupled dynamics of a differential drive system, often leading to overshooting on sharp turns or getting stuck in oscillatory behavior.

### The Solution
To build a robust, agile navigation stack, I architected GlobalDrive using a decoupled, two-layer approach:

1. **Continuous Trajectory Generation:** Instead of standard linear interpolation or cubic splines, I implemented a custom **Quintic Hermite Spline Generator**. This ensures that the generated path is continuous not just in position and velocity, but also in acceleration ($C^2$ continuity), entirely eliminating torque spikes.
2. **Predictive Control:** For the local control loop, I replaced reactive controllers with a **Linear Parameter-Varying Model Predictive Controller (LPV-MPC)** running at 10Hz. By solving a convex optimization problem over a 40-step lookahead horizon, the robot actively anticipates upcoming curves and adjusts its velocity and steering smoothly, constrained by its physical limits.

*(For a deep dive into the mathematics of these algorithms, read my companion post in the [Theory & Notes](/categories/theory/) section).*

### Simulation Results
The stack was rigorously tested against various generated profiles to ensure tracking accuracy and dynamic stability.

*Note: The MPC controller prioritizes smooth, energy-efficient cornering over aggressive, perfect-line tracking based on its current weight tuning.*

**Straight Line & Square Tracking**
![Straight Line](/assets/img/projects/globalDrive/straight_line.png)
![Square](/assets/img/globalDrive/square.png)

**Complex Navigation (ZigZag & Random Waypoints)**
![ZigZag](/assets/img/projects/globalDrive/zigzag.png)
![Random Points](/assets/img/projects/globalDrive/mpc_path.png)

[**Watch the Demo Video Here**](/assets/img/projects/globalDrive/lpv_mpc_demo.webm)