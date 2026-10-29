# Min (@mi1nn)

**Robotics engineer working with ROS 2 — cobot force control, AMR navigation, and simulation-based validation.**

🇰🇷 [한국어](README.ko.md)

---

## About

I studied mechanical engineering, where I worked with structural and vibration analysis. I then worked as a field engineer at an AI healthcare startup, installing our solution in hospitals and maintaining the servers behind it. That job showed me how deployed AI systems actually run, and where they break, in the real world. It also showed me what I wanted to do next: not support finished products, but design the part where hardware and software meet. That is why I moved into robotics.

At the Doosan Robotics **ROKEY AI Bootcamp**, I built ROS 2 robot systems in team projects: force-controlled assembly on a real cobot, a webcam-and-AMR system that tracks a moving target, and an orbital servicing mission simulated in Isaac Sim.

- 🤖 Interests: cobot control, force/torque-based precision assembly, robot vision and 6D pose estimation, simulation-based validation
- 🛠 Currently learning: C++, SLAM, MoveIt 2 motion planning
- 📍 Incheon, South Korea

---

## Projects

| Project | Description | My role | Tech |
|---|---|---|---|
| **[satellite-mission-extension-system](https://github.com/mi1nn/satellite-mission-extension-system)** | Simulates an orbital servicing mission in Isaac Sim. A 7-DOF arm tracks a drifting module with AprilTags, captures it, and docks it to a moving satellite. An Astrobee checks the docking corridor with a point cloud and aborts docking if it is blocked. | Environment setup, docking implementation | Isaac Sim · Isaac Lab · ROS 2 · AprilTag · Point Cloud · FastAPI · Firestore |
| **[overwatch-amr](https://github.com/mi1nn/overwatch-amr)** | A TurtleBot 4 that finds and follows a moving RC car. A fixed webcam locates the car with YOLO, and the robot closes in with Nav2 and tracks it with OAK-D depth. If the car is lost, a state machine spins and searches until it finds it again. | SLAM mapping, YOLO training for the OAK-D camera | ROS 2 · Nav2 · AMCL · SLAM · TurtleBot 4 · OAK-D · YOLO26 |
| **[assembly-cobot](https://github.com/mi1nn/assembly-cobot)** | Assembles solar panel mounting structures with a Doosan cobot. Outdoor light and dust rule out vision, so a three-stage force-controlled insertion absorbs part tolerances. An MES dashboard manages the whole workflow down to robot control. | Project manager, system architecture design | ROS 2 · Doosan M0609 · Force Control · Flask · PostgreSQL |
| **[Project2-ROS2-VLA](https://github.com/mi1nn/Project2-ROS2-VLA)** | A kit assembly robot driven by voice commands. Spoken commands are turned into robot tasks through STT and validation. Object detection and 6D pose estimation tell the robot where to grasp. | Vision: object detection, OctoMap | ROS 2 · MoveIt · YOLO · OctoMap · Whisper · RealSense |

---

## Tech Stack

**Languages** Python · C++ 

**Robotics** ROS 2 (Jazzy) · Nav2 · AMCL · SLAM · tf2 · ros2_control

**Simulation** NVIDIA Isaac Sim · Isaac Lab · USD / PhysX

**AI / Vision** YOLO · OpenCV · AprilTag · OAK-D · RGB-D Point Cloud · OctoMap · PyTorch

**Engineering** CATIA · ANSYS (modal analysis) · Six Sigma GB/BB

**Tools** Git · Docker · PostgreSQL · Linux (Ubuntu) · Bash

---

## Background

- **ROKEY Bootcamp Training Center, Teaching Assistant Intern** — Nov 2026 –
- **Doosan Robotics ROKEY AI Bootcamp** — Completed the ROS 2 cobot and AMR development track
- **AI healthcare startup, CS / Field Engineer** — On-site deployment, server operations, technical support
- **K-Move Future Mobility Quality Management Program (620 hours)** — ISO 9001/14001/45001, IATF 16949, business English
- **B.S. in Mechanical Engineering** — Dynamics, thermodynamics, fluid mechanics, solid mechanics, kinematics, numerical analysis, mechanical vibration
- **Working Holiday, New Zealand** — Customer service in English

---

## Contact

- 📧 alekdi8gm30@gmail.com
