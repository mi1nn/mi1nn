# Overwatch AMR — RC카 추적 로봇

**팀 프로젝트 · 두산로보틱스 ROKEY AI 부트캠프 · 2026.10**
📂 코드: [mi1nn/overwatch-amr](https://github.com/mi1nn/overwatch-amr) · 🇺🇸 [English](overwatch-amr.md) · ← [프로필](../README.ko.md)

<!-- TODO: 데모 GIF 또는 영상 링크 추가 -->

## 개요

고정 웹캠의 도움을 받아 움직이는 RC카를 찾아 따라가는 TurtleBot 4입니다. 웹캠이 YOLO로 RC카를 찾고 호모그래피로 map 좌표를 계산하면, 로봇이 Nav2로 접근한 뒤 OAK-D 카메라로 추적합니다. RC카를 놓치면 제자리에서 돌며 다시 찾습니다.

## 담당

**SLAM 지도 작성, 로봇 OAK-D 카메라용 YOLO 학습**

## 한 일

- **주행용 지도 작성.** TurtleBot 4로 SLAM을 돌려 자동 탐색 지도와 수동 조종 지도 두 개(0.05 m/px)를 만들었고, AMCL과 Nav2가 이 지도를 사용합니다.
- **로봇 카메라로 데이터셋 구축.** OAK-D 영상을 rosbag으로 녹화해 20프레임마다 한 장씩 추출하고, `car`(추적 대상)와 `dummy`(무시할 비슷한 물체) 두 클래스로 라벨링했습니다.
- **YOLO 6개 모델을 같은 조건으로 비교.** YOLOv8n/s, YOLO11n/s, YOLO26n/s를 같은 설정으로 학습하고, 같은 test 셋으로 다시 검증·비교하는 스크립트를 작성했습니다.
- **실패한 학습 원인 분석.** YOLOv8s가 21 epoch에서 조기 종료되며 mAP50-95 0.50, `dummy` 검출 0건이 나왔습니다. 2~3 epoch의 발산 때문에 회복 전에 조기 종료가 걸린 것을 찾아냈고, `patience=50`으로 다시 학습해 같은 발산 뒤 0.95까지 회복되는 것을 확인했습니다.

## 결과

| 모델 | Test mAP50-95 | 추론 속도 |
|---|---:|---:|
| **YOLO26n (선정)** | **0.977** | **3.17 ms/img** |
| 나머지 (v8n/s, 11n/s, 26s) | 0.93 – 0.98 | — |

<p align="center">
  <img src="https://github.com/mi1nn/overwatch-amr/raw/main/docs/images/comparison_graphs.png" width="80%" alt="YOLO 모델 비교">
</p>

> test 셋이 12장(박스 24개)으로 작아서 mAP50-95가 0.01~0.02 차이 나는 것은 통계적으로 의미 있는 차이로 보기 어렵습니다. YOLO26n은 정확도와 함께 로봇에서의 작은 크기와 속도를 고려해 선정했습니다.

## 기술

ROS 2 Jazzy · TurtleBot 4 · SLAM · Nav2 · AMCL · OAK-D · YOLO26 · Ultralytics · Homography
