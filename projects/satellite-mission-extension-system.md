# Automated Satellite Mission Extension System

**Team project · Doosan Robotics ROKEY AI Bootcamp · Sep 2026**
📂 Code: [mi1nn/satellite-mission-extension-system](https://github.com/mi1nn/satellite-mission-extension-system) · 🇰🇷 [한국어](satellite-mission-extension-system.ko.md) · ← [Profile](../README.md)

<p align="center">
  <img src="https://github.com/mi1nn/satellite-mission-extension-system/raw/main/docs/image/mep_%ED%9D%A1%EC%B0%A9.gif" width="49%" alt="Canadarm3 capturing the MEP">
  <img src="https://github.com/mi1nn/satellite-mission-extension-system/raw/main/docs/image/docking.gif" width="49%" alt="MEP docking to the satellite">
</p>

## Overview

An orbital servicing mission simulated in Isaac Sim. A servicer's 7-DOF robot arm (Canadarm3) tracks a drifting Mission Extension Pod (MEP) with AprilTags, captures it, and docks it into the thruster nozzle of a moving satellite. An Astrobee watches the docking corridor with an RGB-D point cloud and stops the docking if the corridor is blocked.

## My role

**Simulation environment setup and docking implementation**

## What I did

- **Built the simulation environment.** Set up Isaac Sim 5.0, Isaac Lab, and a fork of Space Robotics Bench, then assembled the scene: satellite and MEP assets, scale, and camera placement.
- **Implemented the docking sequence end to end.** MRV approach → MEP capture → speed matching with the moving satellite → probe alignment and insertion → gripper release and arm retreat → MRV separation.
- **Adopted a reference-based docking controller.** Brought in a controller that tracks the satellite's *predicted* pose while aligning position and attitude together (`coupled_predictive`), made it the default, and kept the earlier sequential controller (`legacy`) selectable from config.

## Results

| Test (simulation, ground truth) | Result |
|---|---|
| Probe docking to a stationary satellite | 20/20 passed · axial error −9.98 mm, radial 6.53 mm |
| Docking to a drifting satellite (0.02 m/s) | 3/5 succeeded |
| Full 6-DOF mission (capture → docking → separation) | Completed end to end |

> These are records from development runs with the earlier `legacy` controller. A same-condition comparison between `legacy` and `coupled_predictive` in Isaac Sim was not run.

## Limitations

- Capture and docking use a `FixedJoint` approximation, not real latches or suction.
- The satellite's docking pose comes from the simulator, not from vision-based estimation.
- Lateral error on a moving satellite stayed close to the 40 mm coupling limit.

## Tech

Isaac Sim 5.0 · Isaac Lab · Space Robotics Bench · USD / PhysX · ROS 2 Jazzy · AprilTag · DLS IK · RGB-D point cloud · FastAPI · Firestore
