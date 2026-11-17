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

Six models trained with identical settings (100 epochs, patience 20, 640 px, batch 16) and evaluated on the same test set.

| Model | Params (M) | Test mAP50-95 | Latency (ms/img) | |
|---|---:|---:|---:|---|
| **YOLO26n** | **2.50** | **0.977** | **3.17** | ✅ selected |
| YOLO11n | 2.59 | 0.964 | 3.18 | |
| YOLOv8n | 3.01 | 0.949 | 3.48 | |
| YOLO26s | 9.95 | 0.961 | 7.23 | |
| YOLOv8s (retrained, patience 50) | 11.14 | 0.953 | 6.84 | |
| YOLO11s | 9.43 | 0.928 | 6.16 | |
| YOLOv8s (first run) | 11.14 | 0.500 | 7.10 | ❌ diverged, missed every `dummy` |

<p align="center">
  <img src="https://github.com/mi1nn/overwatch-amr/raw/main/results/runs/compare/03_speed_size_vs_accuracy.png" width="90%" alt="AMR YOLO models: accuracy vs speed and size">
  <br><sub>Test mAP50-95 vs latency and parameter count. YOLOv8s (orange) is the first, diverged run.</sub>
</p>

**What the comparison showed**
- **Every properly trained model got all 24 test boxes right**: no misclassification, false positive, or miss between `car` and `dummy`.
- **Small (s) models gave no benefit at this data scale.** They were about 2× slower than the nano (n) models with equal or lower accuracy, and were more prone to diverging early in training.
- **YOLO26n was the best fit for the robot**: the highest mAP50-95 and the lowest latency with the fewest parameters.

> Latency is preprocessing + inference + postprocessing at batch 1 on an RTX 3060 Laptop GPU; it will differ on the robot PC. The test set is small (12 images, 24 boxes), so mAP50-95 differences of 0.01–0.02 are not statistically meaningful.

📄 Full analysis (Korean): [training report](https://github.com/mi1nn/overwatch-amr/blob/main/docs/amr/YOLO_TRAINING_REPORT.md) · [model comparison](https://github.com/mi1nn/overwatch-amr/blob/main/docs/amr/AMR-VISION.md)

## Tech

ROS 2 Jazzy · TurtleBot 4 · SLAM · Nav2 · AMCL · OAK-D · YOLO26 · Ultralytics · Homography
