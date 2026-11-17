# 음성·비전 기반 재난 구조키트 자동 구성 시스템

**팀 프로젝트 · 두산로보틱스 ROKEY AI 부트캠프 · 2026.09**
📂 코드: [mi1nn/Project2-ROS2-VLA](https://github.com/mi1nn/Project2-ROS2-VLA) · 🇺🇸 [English](disaster-kit-assembly.md) · ← [프로필](../README.ko.md)

<p align="center">
  <img src="https://github.com/mi1nn/Project2-ROS2-VLA/raw/main/image/cycle.gif" width="49%" alt="키트 구성 사이클">
  <img src="https://github.com/mi1nn/Project2-ROS2-VLA/raw/main/image/cup.gif" width="49%" alt="컵라면 파지">
</p>

## 개요

음성 명령으로 재난 구조키트를 구성하는 두산 M0609 협동로봇입니다. "키트1번" 같은 음성 명령이 웨이크워드 감지, Whisper STT, GPT-4o를 거쳐 검증된 작업이 됩니다. 로봇은 RGB-D 카메라로 품목을 찾고, MoveIt 2로 충돌 없는 경로를 계획해 키트에 담습니다.

## 담당

**비전: 객체 탐지, OctoMap 충돌 환경**

## 한 일

- **객체 탐지.** YOLO26s-seg 모델로 비전 노드를 만들었습니다. 품목의 3D 위치는 중심 픽셀 하나가 아니라 segmentation 마스크 안 유효 depth의 중위값으로 계산해 노이즈에 강하게 했습니다.
- **2단계 학습.** 9개 품목의 공개 이미지 23,412장으로 1차 학습한 뒤, 실제 작업 환경에서 직접 촬영·라벨링한 159장으로 파인튜닝해 현장의 조명과 카메라 각도에 맞췄습니다.
- **OctoMap 충돌 환경.** 초기 관측 구간의 RealSense 포인트 클라우드로 정적 OctoMap을 만들어 MoveIt 2에 넣어, 계획된 경로가 테이블과 지그를 피하게 했습니다. 입력 필터(1 cm voxel, depth 하한, 밀도 필터)를 조정하고 실행 시 맵을 초기화하도록 했습니다.
- **통합.** 탐지와 OctoMap을 파지 파이프라인에 연결하고 전체 시스템 launch 파일을 작성했습니다.

## 결과

| 파인튜닝 모델 (Validation, 객체 319개) | Box | Mask |
|---|---:|---:|
| Precision | 97.7% | 98.5% |
| Recall | 96.3% | 96.9% |
| mAP50 | 99.0% | 99.0% |
| mAP50-95 | 94.8% | 91.1% |

> 객체 검출·분할 성능이며, 로봇의 파지 성공률이 아닙니다.

## 기술

ROS 2 Jazzy · Doosan M0609 · RealSense · YOLO26-seg · PyTorch · OpenCV · OctoMap · MoveIt 2 · Whisper · Docker
