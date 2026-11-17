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

**Vision: object detection and OctoMap collision environment**

## What I did

- **Object detection.** Built the vision node on a YOLO26s-seg model. For each detected item, the 3D position comes from the median of valid depth values inside its segmentation mask, which is more robust than a single center pixel.
- **Two-stage training.** Trained first on 23,412 public images across 9 item classes, then fine-tuned on 159 images taken and labeled in the actual work cell to adapt to its lighting and camera angle.
- **OctoMap collision environment.** Built a static OctoMap from the RealSense point cloud during an initial observation phase and fed it to MoveIt 2 so planned paths avoid the table and fixtures. Tuned the input filtering (1 cm voxels, depth cutoff, density filter) and reset the map on startup.
- **Integration.** Connected detection and OctoMap with the grasp pipeline and wrote the launch files for the full system.

## Results

| Fine-tuned model (validation, 319 objects) | Box | Mask |
|---|---:|---:|
| Precision | 97.7% | 98.5% |
| Recall | 96.3% | 96.9% |
| mAP50 | 99.0% | 99.0% |
| mAP50-95 | 94.8% | 91.1% |

> These are detection and segmentation scores, not the robot's grasp success rate.

## Tech

ROS 2 Jazzy · Doosan M0609 · RealSense · YOLO26-seg · PyTorch · OpenCV · OctoMap · MoveIt 2 · Whisper · Docker
