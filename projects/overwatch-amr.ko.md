# Overwatch AMR — RC카 추적 로봇

**팀 프로젝트 · 두산로보틱스 ROKEY AI 부트캠프 · 2026.10**
📂 코드: [mi1nn/overwatch-amr](https://github.com/mi1nn/overwatch-amr) · 🇺🇸 [English](overwatch-amr.md) · ← [프로필](../README.ko.md)

<p align="center">
  <img src="../assets/overwatch-amr-demo.gif" width="100%" alt="Overwatch AMR demo">
  <br><sub>최종 시연, 4배속 · 고정 웹캠(왼쪽) · 로봇 OAK-D 카메라(가운데) · RViz 지도(오른쪽)</sub>
</p>

## 개요

고정 웹캠의 도움을 받아 움직이는 RC카를 찾아 따라가는 TurtleBot 4입니다. 웹캠이 YOLO로 RC카를 찾고 호모그래피로 map 좌표를 계산하면, 로봇이 Nav2로 접근한 뒤 OAK-D 카메라로 추적합니다. RC카를 놓치면 제자리에서 돌며 다시 찾습니다.

## 담당

**AMR 비전, 웹캠 호모그래피, depth 카메라 기반 추적 주행**

## 한 일

### 1. AMR 비전 (로봇 OAK-D 카메라)
- **로봇 카메라로 데이터셋 구축.** OAK-D 영상을 rosbag으로 녹화해 20프레임마다 한 장씩 추출하고, `car`(추적 대상)와 `dummy`(무시할 비슷한 물체) 두 클래스로 라벨링했습니다.
- **YOLO 6개 모델을 같은 조건으로 비교**(YOLOv8n/s, YOLO11n/s, YOLO26n/s)하고, 같은 test 셋으로 다시 검증하는 스크립트를 작성해 로봇용 모델로 YOLO26n을 선정했습니다.
- **실패한 학습 원인 분석.** YOLOv8s가 21 epoch에서 조기 종료되며 mAP50-95 0.50, `dummy` 검출 0건이 나왔습니다. 2~3 epoch의 발산 때문에 회복 전에 조기 종료가 걸린 것을 찾아냈고, `patience=50`으로 다시 학습해 같은 발산 뒤 0.95까지 회복되는 것을 확인했습니다.

### 2. 웹캠 호모그래피 (픽셀 → map 좌표)
- 고정 웹캠은 픽셀 좌표만 알기 때문에, 바닥 위 여러 점의 픽셀 위치와 map 위치 쌍으로 호모그래피 **H**를 구했습니다. map 위치는 로봇의 AMCL pose(`/amcl_pose`)나 RViz에서 얻었습니다.
- 탐지된 RC카는 **bbox 하단 중앙**, 즉 바닥에 닿는 점을 **H**로 변환합니다. 호모그래피는 바닥 평면에서만 맞기 때문에, bbox 중심을 쓰면 차 높이만큼 위치가 밀립니다.
- 보정 노드는 점별 재투영 오차, RANSAC 인라이어, leave-one-out 오차를 출력해 주행 전에 잘못된 보정을 걸러낼 수 있게 했습니다.

### 3. 추적 주행 (depth 카메라 + AMCL pose + Nav2)
로봇 카메라에 RC카가 보이면, RC카의 map 위치를 계산해 일정 거리를 유지하며 따라갑니다. 막힌 시험 공간의 지도는 SLAM으로 직접 만들어 저장했습니다(자동 탐색, 수동 조종 두 가지, 0.05 m/px). 실제 추적할 때는 지도를 새로 만들지 않고, AMCL이 2D 라이다로 저장된 지도 위 로봇 위치를 추정하며, 같은 라이다가 Nav2 costmap을 갱신해 추적 중 경로상의 장애물을 감지합니다.

1. **영상에서 방향 계산.** bbox의 가로 위치와 OAK-D 수평 화각(69°)으로 RC카까지의 방향각을 구합니다.
2. **depth로 거리 계산.** bbox 중심의 작은 depth 영역 중앙값으로 광축 방향 거리를 구하고, 방향각으로 보정해 실제 거리를 얻습니다.
3. **RC카의 map 좌표 계산.** 방향각과 거리를 AMCL로 추정한 로봇 pose(`map → base_link`)와 합쳐 RC카의 map 좌표를 구합니다.
4. **Nav2로 1.0 m 유지하며 추종.** RC카 위치를 Nav2 목표로 보내고, 경로상 1.0 m가 남으면 멈추게 해 Nav2가 벽을 돌아가는 경로를 그대로 쓸 수 있게 했습니다. 목표는 최소 0.5초 간격으로, RC카가 0.2 m 이상 움직였을 때만 다시 보내 경로를 계속 재계획하지 않게 했습니다.
5. **가까우면 화면 중앙 유지.** 1.0 m 안으로 들어오면 목표를 취소하고, P 제어(게인 1.2, 최대 1.0 rad/s)로 제자리 회전해 RC카를 영상 중앙에 둡니다.

## 결과

6개 모델을 같은 설정(100 epoch, patience 20, 640 px, batch 16)으로 학습하고 같은 test 셋으로 평가했습니다.

| 모델 | 파라미터 (M) | Test mAP50-95 | 지연 시간 (ms/img) | |
|---|---:|---:|---:|---|
| **YOLO26n** | **2.50** | **0.977** | **3.17** | ✅ 선정 |
| YOLO11n | 2.59 | 0.964 | 3.18 | |
| YOLOv8n | 3.01 | 0.949 | 3.48 | |
| YOLO26s | 9.95 | 0.961 | 7.23 | |
| YOLOv8s (재학습, patience 50) | 11.14 | 0.953 | 6.84 | |
| YOLO11s | 9.43 | 0.928 | 6.16 | |
| YOLOv8s (1차) | 11.14 | 0.500 | 7.10 | ❌ 발산, `dummy` 전부 미검출 |

<p align="center">
  <img src="https://github.com/mi1nn/overwatch-amr/raw/main/results/runs/compare/03_speed_size_vs_accuracy.png" width="90%" alt="AMR YOLO 모델 정확도 대비 속도·크기">
  <br><sub>지연 시간·파라미터 수 대비 test mAP50-95. 주황색 YOLOv8s는 발산한 1차 학습 결과입니다.</sub>
</p>

**비교로 확인한 것**
- **정상 학습된 모델은 모두 test 박스 24개를 전부 맞혔습니다.** `car`와 `dummy` 사이 오분류, 오탐, 미탐이 하나도 없었습니다.
- **이 데이터 규모에서는 s 모델의 이점이 없었습니다.** n 모델보다 약 2배 느리면서 정확도는 같거나 낮았고, 학습 초반에 발산하기도 쉬웠습니다.
- **로봇에는 YOLO26n이 가장 적합했습니다.** mAP50-95가 가장 높고, 파라미터가 가장 적으면서 지연 시간도 가장 짧았습니다.

> 지연 시간은 RTX 3060 Laptop GPU에서 batch 1로 잰 전처리+추론+후처리 합계이며, 로봇 PC에서는 달라집니다. test 셋이 12장(박스 24개)으로 작아서 mAP50-95가 0.01~0.02 차이 나는 것은 통계적으로 의미 있는 차이로 보기 어렵습니다.

📄 전체 분석: [학습 통계 보고서](https://github.com/mi1nn/overwatch-amr/blob/main/docs/amr/YOLO_TRAINING_REPORT.md) · [모델 비교](https://github.com/mi1nn/overwatch-amr/blob/main/docs/amr/AMR-VISION.md)

## 기술

ROS 2 Jazzy · TurtleBot 4 · 2D LiDAR · SLAM (지도 작성) · AMCL · Nav2 · OAK-D · YOLO26 · Ultralytics · Homography
