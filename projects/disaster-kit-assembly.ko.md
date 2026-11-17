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

**음성 패키지(`kit_voice`), 비전 패키지(`kit_vision`), OctoMap 장애물 환경**

## 한 일

### 1. `kit_voice` — 음성 명령을 검증된 작업으로
- **웨이크워드.** openWakeWord로 "hello rokey"를 감지하고, 마이크를 80 ms 단위로 읽습니다. STT를 시작하기 전에 마이크를 놓아 STT가 같은 장치를 열 수 있게 했고, USB 마이크가 빠지거나 늦게 연결되면 자동으로 다시 엽니다.
- **음성 인식(STT).** 16 kHz로 5초 녹음해 Whisper API로 보냅니다. 16 kHz 입력 장치를 자동으로 고르고, 다른 환경을 위해 직접 지정할 수도 있게 했습니다.
- **명령 해석과 검증.** GPT-4o(LangChain)로 문장을 키트 이름·품목·수량 JSON으로 바꿉니다. 프롬프트는 품목 목록과 키트 레시피(예: "키트1번" → 컵라면, 샴푸리필, 양갱)로 만듭니다. 결과는 로봇에 넘기기 전에 검사해서, 지원하지 않는 품목이나 잘못된 수량, JSON이 아닌 응답은 거부합니다. 품목 목록은 `kit_vision`의 클래스 파일을 그대로 읽어서 음성과 비전의 품목 이름이 항상 같게 했습니다.
- **서비스 인터페이스.** `/get_command` 서비스로 제공합니다. OpenAI 크레딧 소진과 요청 한도 초과는 대응 방법이 달라서(충전 vs 재시도) 서로 다른 오류로 알립니다.

### 2. `kit_vision` — 객체 탐지에서 3D 위치까지
- **카메라 입력.** RealSense 컬러, 정렬된 depth, 카메라 정보를 구독하되 최신 프레임만 유지하고, 컬러와 depth의 시각 차이가 큰 쌍은 버립니다.
- **추론 루프.** YOLO segmentation 모델을 별도 스레드에서 약 3 Hz로 돌립니다. PyTorch가 CPU 스레드를 10~16개씩 잡아 RealSense USB 드라이버가 밀리는 문제가 있어서, 스레드 수를 제한했습니다.
- **3D 위치 계산.** 품목마다 **segmentation 폴리곤 안 유효 depth의 중앙값**을 써서 중심 픽셀 하나보다 노이즈에 강하게 했고, 카메라 내부 파라미터로 카메라 좌표로 변환합니다. 클래스, 점수, 위치, 마스크와 디버그 영상을 함께 발행합니다.
- **컨테이너.** GPU 추론을 Docker로 패키징했습니다.

### 3. MoveIt 2용 OctoMap 장애물 환경
- **포인트 클라우드 필터링.** RealSense 포인트 클라우드 → 잘못된 점 제거 → 0.28 m보다 가까운 점 제거 → 로봇 베이스 좌표로 변환 → 1 cm voxel → 주변 27칸 중 4칸 이상 차 있는 voxel만 유지. 떨어져 있는 depth 노이즈는 없애고, 얇은 상자 벽과 모서리는 남깁니다.
- **정적 지도.** 관찰 자세에서 3초간 스캔한 뒤 지도를 고정해 작업 사이에 재사용합니다. 작업 중에 경로 계획이 느려지거나 바뀌지 않게 하기 위해서이고, 실행할 때마다 지도를 초기화합니다.
- **충돌 허용 설정.** 그리퍼와 손목 링크는 OctoMap과 닿아도 되게 했습니다. 그렇지 않으면 그리퍼 자체에 찍힌 점 때문에 모든 경로 계획이 막힙니다. 이 설정은 *현재* 충돌 허용 행렬에 합쳐서 보냅니다. 일부만 보내면 행렬 전체가 바뀌어 로봇의 자기 충돌 규칙이 조용히 사라지기 때문입니다.

## 결과

아래 탐지 모델은 팀원이 학습했고, 제 `kit_vision` 노드가 파이프라인에서 이 모델을 실행합니다.

| 파인튜닝 모델 (Validation, 객체 319개) | Box | Mask |
|---|---:|---:|
| Precision | 97.7% | 98.5% |
| Recall | 96.3% | 96.9% |
| mAP50 | 99.0% | 99.0% |
| mAP50-95 | 94.8% | 91.1% |

> 객체 검출·분할 성능이며, 로봇의 파지 성공률이 아닙니다.

## 기술

ROS 2 Jazzy · Doosan M0609 · RealSense · YOLO26-seg · PyTorch · OpenCV · OctoMap · MoveIt 2 · Whisper · Docker
