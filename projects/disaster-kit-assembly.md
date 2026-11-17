# Voice & Vision Disaster Relief Kit Assembly

**Team project (4 people) · Doosan Robotics ROKEY AI Bootcamp · Sep 2026**
📂 Code: [mi1nn/voice-controlled-kit-assembly](https://github.com/mi1nn/voice-controlled-kit-assembly) · 🇰🇷 [한국어](disaster-kit-assembly.ko.md) · ← [Profile](../README.md)

<p align="center">
  <img src="https://github.com/mi1nn/voice-controlled-kit-assembly/raw/main/image/cycle.gif" width="49%" alt="Kit assembly cycle">
  <img src="https://github.com/mi1nn/voice-controlled-kit-assembly/raw/main/image/cup.gif" width="49%" alt="Cup noodle grasp">
</p>

## Overview

A Doosan M0609 cobot that packs disaster relief kits from voice commands. A spoken order ("kit 1") goes through wake-word detection, Whisper STT, and GPT-4o into a validated task. The robot then finds each item with an RGB-D camera, plans a collision-free path with MoveIt 2, and places it in the kit.

## My role

**System and ROS 2 interface design, voice package (`kit_voice`), vision package (`kit_vision`) with initial position estimation, OctoMap obstacle environment, and system integration**

## What I did

### 1. System and ROS 2 interface design
- **Interface contracts first.** Defined the topics, services, and messages between voice, vision, controller, motion, and database in a shared `kit_interfaces` package, so each package depends on the contract rather than on another package's code.
- **Work in parallel.** With the contracts fixed early, a mock detection publisher let the team wire up the coordinate pipeline before the model and the robot were ready.

### 2. `kit_voice` — voice command to a validated task
- **Wake word.** Listens for "hello rokey" with openWakeWord, reading the mic in 80 ms frames. Before speech-to-text starts, the node releases the mic so STT can open the same device, and it reopens the mic automatically if a USB mic drops or connects late.
- **Speech to text.** Records 5 s at 16 kHz and sends it to the Whisper API. A 16 kHz input device is picked automatically, with an override for other setups.
- **Command parsing with validation.** GPT-4o (via LangChain) turns the sentence into JSON — kit name, items, quantities — using a prompt built from the item list and kit recipes (e.g. "kit 1" → cup noodle, shampoo refill, yanggaeng). The result is checked before it reaches the robot: unsupported items, invalid quantities, or non-JSON replies are rejected. The item list is read from `kit_vision`'s class file, so voice and vision always agree on item names.
- **Service interface.** Exposed as the `/get_command` service. OpenAI credit exhaustion and rate limits are reported as separate errors, because one needs a top-up and the other only a retry.

### 3. `kit_vision` — object detection to 3D positions
- **Camera input.** Subscribes to RealSense color, aligned depth, and camera info, keeping only the latest frame, and drops color/depth pairs whose timestamps are too far apart.
- **Inference loop.** Runs the YOLO segmentation model in its own thread at about 3 Hz. PyTorch was grabbing 10–16 CPU threads and starving the RealSense USB driver, so I capped its thread count.
- **3D position.** For each item, the depth is the **median of valid depth values inside its segmentation polygon** — more robust than one center pixel — and is converted to camera coordinates with the camera intrinsics. Results are published with class, score, position, and mask, plus a debug image.
- **Initial position estimation.** Converted camera-frame positions to the robot's base frame with the hand–eye calibration, and added a timestamp check so the robot never acts on a stale detection.
- **Container.** Packaged GPU inference in Docker.

### 4. OctoMap obstacle environment for MoveIt 2
- **Point cloud filtering.** RealSense point cloud → drop invalid points → drop points closer than 0.28 m → transform to the robot base → 1 cm voxels → keep a voxel only if at least 4 of its 27 neighbors are occupied. This removes isolated depth noise while keeping thin box walls and corners.
- **Static map.** The robot scans for 3 s from an observation pose, then the map is frozen and reused across tasks, so planning does not slow down or change mid-task. The map is cleared on startup.
- **Iterated to a narrower scope.** Live OctoMap kept marking the arm, the gripper, and tray-surface noise as obstacles, so plans failed often. I tuned voxel size, RealSense filters, and target masking through 16 versions, and finally limited the map to the inspection area as a static map.
- **Allowed collisions.** The gripper and wrist links are allowed to touch the OctoMap; otherwise points on the gripper itself block every plan. This is merged into the *current* allowed-collision matrix, because sending a partial matrix replaces the whole one and silently drops the robot's own self-collision rules.

### 5. System integration
- Connected the end-to-end flow — voice → vision → position estimation → controller → motion → inspection → database — and ran the full system on the real robot with the team.

## Results

### First end-to-end evaluation (Sep 8, 2026 · v0.1.0)
22 runs on the real robot, with 8–9 items placed randomly in the pick area and a box placed on the kit tray as an obstacle. Results were analyzed from the MongoDB execution logs; because the gripper's own success flag was unreliable, grasp results were labeled by hand from video.

| Metric | Result |
|---|---:|
| **Command keyword recognition** (voice) | **20/22 (90.9%)** |
| **Valid JSON command** (LLM + validation) | **22/22 (100%)** |
| Item pick-and-place success | 84/116 (72.4%) |
| Kit composition (items actually in the kit) | 78/112 (69.6%) |
| Kits fully assembled | 3/22 (13.6%) |
| **End-to-end success** (assembled *and* final inspection PASS) | **0/22 (0%)** |
| Longest continuous run | 1 h 13 min |

- **Voice side (my `kit_voice`)**: every recognized command became valid JSON. The two misses were speech-recognition errors on the kit name ("응급기트", "시사 킬투"), not parsing errors.
- **Where it failed**: the bottleneck was grasping and the final inspection, not perception of commands. Grasp success varied widely by item — ham 86.7%, shampoo refill and spoon 85.7%, but cup noodle 53.3% and mask 16.7% — showing that a 2D mask-and-depth grasp does not fit every shape.

<details>
<summary><strong>Grasp success by item</strong></summary>

| Item | Success | Item | Success |
|---|---:|---|---:|
| Ham | 13/15 (86.7%) | Yanggaeng | 14/20 (70.0%) |
| Shampoo refill | 6/7 (85.7%) | Travel tissue | 6/9 (66.7%) |
| Disposable spoon | 12/14 (85.7%) | Cup noodle | 8/15 (53.3%) |
| Scrubber | 13/16 (81.3%) | Mask | 1/6 (16.7%) |
| Formula milk | 11/14 (78.6%) | | |

</details>

### What changed after the evaluation (v0.2.0)
- **Motion:** fixed Cartesian paths and joint goals failed repeatedly, so picking and observation moved to **MoveIt 2 pose goals** with collision checking.
- **OctoMap:** applied to the inspection area as the static map described above. Applying it to the pick and observe areas was dropped because point-cloud noise kept creating false obstacles.
- **Grasping:** cup noodles got a 6-DoF grasp pipeline (FoundationPose + GraspGenX) with up to 5 candidates and 3 re-observations.

> **Limitation:** the planned second evaluation (20 tasks) was not run, so no improvement over the first evaluation is claimed.

### Detection model
Trained by a teammate; my `kit_vision` node runs it in the pipeline.

| Fine-tuned YOLO26s-seg (validation, 319 objects) | Box | Mask |
|---|---:|---:|
| Precision | 97.7% | 98.5% |
| Recall | 96.3% | 96.9% |
| mAP50 | 99.0% | 99.0% |
| mAP50-95 | 94.8% | 91.1% |

> These are detection scores, not grasp success rates.

### Demo videos
[Demo 1](https://youtu.be/gEU7pFmAMOg) · [Demo 2](https://youtu.be/UJp21P18yG4) · [Demo 3](https://youtu.be/BEMkkk1yyE4) · [Demo 4](https://youtu.be/7j6hGa2-f7E)

## Tech

ROS 2 Jazzy · Doosan M0609 · RealSense · YOLO26-seg · PyTorch · OpenCV · OctoMap · MoveIt 2 · Whisper · Docker
