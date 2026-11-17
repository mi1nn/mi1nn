# Overwatch AMR — RC Car Chasing Robot

**Team project · Doosan Robotics ROKEY AI Bootcamp · Oct 2026**
📂 Code: [mi1nn/overwatch-amr](https://github.com/mi1nn/overwatch-amr) · 🇰🇷 [한국어](overwatch-amr.ko.md) · ← [Profile](../README.md)

<p align="center">
  <img src="../assets/overwatch-amr-demo.gif" width="100%" alt="Overwatch AMR demo">
  <br><sub>Final demo, 4× speed · Fixed webcam (left) · Robot OAK-D camera (center) · RViz map (right)</sub>
</p>

## Overview

A TurtleBot 4 that finds and follows a moving RC car with help from a fixed webcam. The webcam locates the car with YOLO and converts it to map coordinates through a homography. The robot closes in with Nav2, then tracks the car with its OAK-D camera, and spins to search if it loses the car.

## My role

**SLAM mapping and YOLO training for the robot's OAK-D camera**

## What I did

- **Built the maps for navigation.** Created two maps with SLAM on TurtleBot 4 — one by autonomous exploration and one by manual driving (0.05 m/px) — used by AMCL and Nav2.
- **Built the dataset from the robot's own camera.** Recorded rosbags from the OAK-D, extracted one frame every 20, and labeled two classes: `car` (target) and `dummy` (look-alike to ignore).
- **Compared six YOLO models under the same conditions.** Trained YOLOv8n/s, YOLO11n/s, and YOLO26n/s with identical settings and wrote scripts to re-validate and compare them on the same test set.
- **Diagnosed a failed training run.** YOLOv8s stopped early at epoch 21 with mAP50-95 0.50 and no `dummy` detections. I traced it to divergence in epochs 2–3 triggering early stopping before recovery; retraining with `patience=50` reproduced the divergence but recovered to 0.95.

## Results

| Model | Test mAP50-95 | Inference |
|---|---:|---:|
| **YOLO26n (selected)** | **0.977** | **3.17 ms/img** |
| Others (v8n/s, 11n/s, 26s) | 0.93 – 0.98 | — |

<p align="center">
  <img src="https://github.com/mi1nn/overwatch-amr/raw/main/results/runs/compare/03_speed_size_vs_accuracy.png" width="90%" alt="AMR YOLO models: accuracy vs speed and size">
  <br><sub>Robot (OAK-D) models: test mAP50-95 vs inference time and parameter count. YOLOv8s (orange) is the first, diverged run; retraining reached 0.95.</sub>
</p>

> The test set is small (12 images, 24 boxes), so differences of 0.01–0.02 in mAP50-95 are not statistically meaningful. YOLO26n was chosen for its accuracy *and* small size and speed on the robot.

## Tech

ROS 2 Jazzy · TurtleBot 4 · SLAM · Nav2 · AMCL · OAK-D · YOLO26 · Ultralytics · Homography
