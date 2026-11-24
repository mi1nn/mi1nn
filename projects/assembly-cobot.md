# Assembly-Cobot — Solar Panel Structure Assembly

**Team project (4 people) · Doosan Robotics ROKEY AI Bootcamp · Aug 2026**
📂 Code: [mi1nn/assembly-cobot](https://github.com/mi1nn/assembly-cobot) · 🇰🇷 [한국어](assembly-cobot.ko.md) · ← [Profile](../README.en.md)

<p align="center">
  <img src="https://github.com/mi1nn/assembly-cobot/raw/main/img/01_post.gif" width="49%" alt="Column insertion">
  <img src="https://github.com/mi1nn/assembly-cobot/raw/main/img/03_snapfit.gif" width="49%" alt="Snap pin fastening">
</p>

## Overview

A Doosan M0609 cobot assembles a solar panel mounting structure: columns → frames → snap pins → panel. Outdoor light and dust make vision unreliable, so the system uses no vision at all. Instead, F/T-sensor force control absorbs part tolerances during insertion.

## My role

**Project manager and system architecture design**

## What I did

- **Led the project.** Set the scope with the team and kept requirements, test plans, and the final scenario in ten requirement documents (business, system, function, interface, test, hardware, DB, ops, maintenance).
- **Designed the system architecture.** A layered stack: web dashboard → Flask backend → PostgreSQL → ROS 2 bridge → robot control node. The robot never writes to the database directly; the bridge is the only path between them.
- **Designed the execution model.** A three-level hierarchy — work order → work execution → operation execution — so every robot step maps to a database record. One ROS 2 action goal runs one work execution, and its feedback updates each operation.
- **Worked on the robot side.** Refined the force-control routine and changed the action interface so the components to install come from the database instead of being hard-coded.

## How the force control works

- **Two-step insertion:** first insertion, then a second press to seat the part. If the first insertion fails, the robot searches 25 points on a 0.3 mm grid for the best position and retries.
- **Periodic fitting:** a small XY vibration seats frames on columns and aligns the panel.
- **Arc paths:** quadratic paths move parts from low pickup positions to high install positions without contact.

## Tech

ROS 2 Jazzy · Doosan M0609 · DRL · F/T force control · Flask · SQLAlchemy · PostgreSQL
