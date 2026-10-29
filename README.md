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
| **[satellite-mission-extension-system](https://github.com/mi1nn/satellite-mission-extension-system)** | Orbital servicing mission simulated in Isaac Sim. A servicer's 7-DOF robot arm captures a drifting Mission Extension Pod (MEP), docks it to a moving satellite, and releases it. AprilTags on the wrist camera give the capture-point pose, which is predicted ahead and tracked with DLS IK. An Astrobee builds an RGB-D point cloud of the docking corridor and aborts docking before insertion if it finds an obstacle. Mission state streams over ROS 2 to a live dashboard and is logged to Firestore for replay with synced video. | Environment setup, docking implementation | Isaac Sim · Isaac Lab · ROS 2 · AprilTag · Point Cloud · FastAPI · Firestore |
| **[overwatch-amr](https://github.com/mi1nn/overwatch-amr)** | AMR that finds and follows a moving RC car with help from a fixed webcam. The webcam PC detects the car with YOLO and maps it to map coordinates through a homography. The TurtleBot 4 approaches with Nav2, then follows the car using the OAK-D bounding-box bearing and depth, avoiding obstacles. An approach → chase → search state machine recovers when the target is lost. Six YOLO models (v8/11/26) were compared under the same conditions; YOLO26n was selected (mAP50-95 0.977, 3.2 ms/img). | SLAM mapping, YOLO training for the OAK-D camera | ROS 2 · Nav2 · AMCL · SLAM · TurtleBot 4 · OAK-D · YOLO26 |
| **[assembly-cobot](https://github.com/mi1nn/assembly-cobot)** | Automated assembly of solar panel mounting structures with F/T-sensor force control. Outdoor lighting and dust make vision unreliable, so the system uses no vision at all. Instead, a three-stage force-controlled pin insertion (first insertion → re-grasp → final insertion) absorbs part tolerances. The full stack runs from an MES dashboard down to the robot control node. | Project manager, system architecture design | ROS 2 · Doosan M0609 · Force Control · Flask · PostgreSQL |
| **[Project2-ROS2-VLA](https://github.com/mi1nn/Project2-ROS2-VLA)** | Kit assembly robot driven by voice commands. Spoken commands go through STT, keyword extraction, and validation, and become robot tasks. RealSense-based object detection and 6D pose estimation decide where to grasp. | Vision: object detection, OctoMap | ROS 2 · MoveIt · YOLO · OctoMap · Whisper · RealSense |

---

## Tech Stack

**Robotics** ROS 2 (Jazzy) · Nav2 · AMCL · SLAM · tf2 · TurtleBot 4 · Doosan Robotics API · ros2_control · OnRobot Gripper

**Simulation** NVIDIA Isaac Sim · Isaac Lab · USD / PhysX

**AI / Vision** YOLO · OpenCV · AprilTag · Intel RealSense · OAK-D · RGB-D Point Cloud · OctoMap · Whisper (STT) · PyTorch

**Backend** Python · FastAPI · PostgreSQL · Docker

**Engineering** CATIA · ANSYS (modal analysis) · Six Sigma GB/BB · ISO 9001 / IATF 16949

**Tools** Git · Linux (Ubuntu) · Bash

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
