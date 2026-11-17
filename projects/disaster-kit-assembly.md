# Voice & Vision Disaster Relief Kit Assembly

**Team project · Doosan Robotics ROKEY AI Bootcamp · Sep 2026**
📂 Code: [mi1nn/Project2-ROS2-VLA](https://github.com/mi1nn/Project2-ROS2-VLA) · 🇰🇷 [한국어](disaster-kit-assembly.ko.md) · ← [Profile](../README.md)

<p align="center">
  <img src="https://github.com/mi1nn/Project2-ROS2-VLA/raw/main/image/cycle.gif" width="49%" alt="Kit assembly cycle">
  <img src="https://github.com/mi1nn/Project2-ROS2-VLA/raw/main/image/cup.gif" width="49%" alt="Cup noodle grasp">
</p>

## Overview

A Doosan M0609 cobot that packs disaster relief kits from voice commands. A spoken order ("kit 1") goes through wake-word detection, Whisper STT, and GPT-4o into a validated task. The robot then finds each item with an RGB-D camera, plans a collision-free path with MoveIt 2, and places it in the kit.

## My role

**Voice package (`kit_voice`), vision package (`kit_vision`), and the OctoMap obstacle environment**

## What I did

### 1. `kit_voice` — voice command to a validated task
- **Wake word.** Listens for "hello rokey" with openWakeWord, reading the mic in 80 ms frames. Before speech-to-text starts, the node releases the mic so STT can open the same device, and it reopens the mic automatically if a USB mic drops or connects late.
- **Speech to text.** Records 5 s at 16 kHz and sends it to the Whisper API. A 16 kHz input device is picked automatically, with an override for other setups.
- **Command parsing with validation.** GPT-4o (via LangChain) turns the sentence into JSON — kit name, items, quantities — using a prompt built from the item list and kit recipes (e.g. "kit 1" → cup noodle, shampoo refill, yanggaeng). The result is checked before it reaches the robot: unsupported items, invalid quantities, or non-JSON replies are rejected. The item list is read from `kit_vision`'s class file, so voice and vision always agree on item names.
- **Service interface.** Exposed as the `/get_command` service. OpenAI credit exhaustion and rate limits are reported as separate errors, because one needs a top-up and the other only a retry.

### 2. `kit_vision` — object detection to 3D positions
- **Camera input.** Subscribes to RealSense color, aligned depth, and camera info, keeping only the latest frame, and drops color/depth pairs whose timestamps are too far apart.
- **Inference loop.** Runs the YOLO segmentation model in its own thread at about 3 Hz. PyTorch was grabbing 10–16 CPU threads and starving the RealSense USB driver, so I capped its thread count.
- **3D position.** For each item, the depth is the **median of valid depth values inside its segmentation polygon** — more robust than one center pixel — and is converted to camera coordinates with the camera intrinsics. Results are published with class, score, position, and mask, plus a debug image.
- **Container.** Packaged GPU inference in Docker.

### 3. OctoMap obstacle environment for MoveIt 2
- **Point cloud filtering.** RealSense point cloud → drop invalid points → drop points closer than 0.28 m → transform to the robot base → 1 cm voxels → keep a voxel only if at least 4 of its 27 neighbors are occupied. This removes isolated depth noise while keeping thin box walls and corners.
- **Static map.** The robot scans for 3 s from an observation pose, then the map is frozen and reused across tasks, so planning does not slow down or change mid-task. The map is cleared on startup.
- **Allowed collisions.** The gripper and wrist links are allowed to touch the OctoMap; otherwise points on the gripper itself block every plan. This is merged into the *current* allowed-collision matrix, because sending a partial matrix replaces the whole one and silently drops the robot's own self-collision rules.

## Results

The detection model below was trained by a teammate; my `kit_vision` node runs it in the pipeline.

| Fine-tuned model (validation, 319 objects) | Box | Mask |
|---|---:|---:|
| Precision | 97.7% | 98.5% |
| Recall | 96.3% | 96.9% |
| mAP50 | 99.0% | 99.0% |
| mAP50-95 | 94.8% | 91.1% |

> These are detection and segmentation scores, not the robot's grasp success rate.

## Tech

ROS 2 Jazzy · Doosan M0609 · RealSense · YOLO26-seg · PyTorch · OpenCV · OctoMap · MoveIt 2 · Whisper · Docker
