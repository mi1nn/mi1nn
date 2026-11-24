# Automated Satellite Mission Extension System

**Team project (3 people) · Doosan Robotics ROKEY AI Bootcamp · Sep 2026**
📂 Code: [mi1nn/satellite-mission-extension-system](https://github.com/mi1nn/satellite-mission-extension-system) · 🇰🇷 [한국어](satellite-mission-extension-system.ko.md) · ← [Profile](../README.en.md)

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
- **Adopted a reference-based docking controller.** Brought in a controller that tracks the satellite's *predicted* pose while aligning position and attitude together (`coupled_predictive`), made it the default, and kept the earlier sequential controller (`legacy`) selectable from config. Details [below](#reference-based-docking-controller).

## Reference-based docking controller

### Why it was needed
The arm carries a 3,000 kg MEP, which makes the arm + payload swing slowly with a period of about 20 s and very little damping. The earlier `legacy` controller avoided exciting this swing by moving very slowly and aligning in sequence (position first, then attitude). On a drifting satellite, its lateral docking error ended up at 39.97–39.98 mm, right at the 40 mm coupling limit.

### Papers applied

| Reference | Idea taken | How it was applied |
|---|---|---|
| Zhou, Liu, Cai — *Motion-planning and pose-tracking based rendezvous and docking with a tumbling target* | Separate *where the dock will be* from *how to track it* | Predict the dock pose 0.3 s ahead from the satellite's linear and angular velocity (constant-twist model), and track that predicted pose |
| Ye, Lu, Mu — *Compound control for autonomous docking to a three-axis tumbling target* | Track position, attitude, and relative linear/angular velocity together | Feed-forward + P + D tracker that aligns position and attitude at the same time; attitude error as an SO(3) rotation vector $e_R=\log(R_dR_c^T)$ |
| Chen & Sun (2020) — *Nonlinear control of underactuated systems subject to both actuated and unactuated state constraints* | Continuous constraints instead of on/off switching | A soft gate that slows the approach in proportion to misalignment (10–80 mm, 1–7°) instead of a hard stop, plus hysteresis on rollback |
| Singer & Seering (1990) — input shaping | Step commands excite residual vibration | Smooth velocity ramps instead of velocity steps |

**Prediction and tracking**

$$p_d(t+T)=c+v_cT+\exp([\omega]T)\,(p_d(t)-c)$$
$$v_{cmd}=v_D+\operatorname{clip}(K_p e_p)-K_d(v_{tip}-v_D)$$

**Alignment check:** insertion starts only when lateral error < 50 mm, attitude error < 4°, relative speed < 0.02 m/s, and relative rotation < 1°/s all hold for 0.5 s straight. An emergency stop triggers above 150 mm or 12°.

### Considered but not adopted
- **Sliding-mode control** (from Ye et al.): its switching could excite the 20 s swing mode.
- **Heavy-payload impedance control** (Aghili, 2009): active damping with gain 1.0 diverged within 0.1 s, and no stable gain was found.
- **Reaction null-space** (Nenchev et al., 1999) and **multi-mode input shaping** (Singhose, 2009): the base is fixed and only one swing mode dominates, so they did not address the actual problem.
- **Flatness-based trajectory planning:** it assumes a precomputed trajectory, which does not fit closed-loop docking that re-reads the drifting target every step.

### Tuning: alignment was too slow
In the first Isaac Sim run with the new controller, attitude converged quickly but lateral error closed at only about 7 mm/s, so alignment would take about 50 s more. From the control law, when the error is large the P term saturates at its limit while the D term subtracts the motion it causes, so speed settles at **limit ÷ (1 + K_d)** = 15 ÷ 1.6 ≈ 9.4 mm/s — consistent with the toy model (8.0 mm/s) and the simulator (≈ 7 mm/s). Comparing limits of 15–30 mm/s in an offline toy model, raising the limit to 0.03 m/s cut alignment time from 41 s to 27 s with the least overshoot.

### Validation status
- Offline unit tests cover prediction, SO(3) error, gates, hysteresis, and velocity estimation.
- The 41 s → 27 s result comes from the offline toy model. **A same-condition comparison of `legacy` vs `coupled_predictive` in Isaac Sim was not completed**, so no improvement in docking success or error is claimed.

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
