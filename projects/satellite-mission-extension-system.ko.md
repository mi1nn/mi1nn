# 위성 수명연장 임무 자동화 시스템

**팀 프로젝트 · 두산로보틱스 ROKEY AI 부트캠프 · 2026.09**
📂 코드: [mi1nn/satellite-mission-extension-system](https://github.com/mi1nn/satellite-mission-extension-system) · 🇺🇸 [English](satellite-mission-extension-system.md) · ← [프로필](../README.ko.md)

<p align="center">
  <img src="https://github.com/mi1nn/satellite-mission-extension-system/raw/main/docs/image/mep_%ED%9D%A1%EC%B0%A9.gif" width="49%" alt="Canadarm3의 MEP 포획">
  <img src="https://github.com/mi1nn/satellite-mission-extension-system/raw/main/docs/image/docking.gif" width="49%" alt="MEP의 위성 도킹">
</p>

## 개요

Isaac Sim에서 궤도 위성 서비싱 임무를 시뮬레이션했습니다. 서비스 위성의 7자유도 로봇팔(Canadarm3)이 표류하는 수명연장 모듈(MEP)을 AprilTag로 추적해 포획하고, 이동 중인 위성의 추력기 노즐에 도킹합니다. Astrobee는 RGB-D 포인트 클라우드로 도킹 통로를 확인해 장애물이 있으면 도킹을 중단합니다.

## 담당

**시뮬레이션 환경 세팅, 도킹 구현**

## 한 일

- **시뮬레이션 환경 구축.** Isaac Sim 5.0, Isaac Lab, Space Robotics Bench 포크를 세팅하고, 위성·MEP 자산과 크기, 카메라 배치를 맞춰 장면을 구성했습니다.
- **도킹 시퀀스 전체 구현.** MRV 접근 → MEP 포획 → 이동 위성과 속도 정합 → probe 정렬·삽입 → 그리퍼 해제와 팔 후퇴 → MRV 분리까지 이어지는 흐름을 구현했습니다.
- **레퍼런스 기반 도킹 제어기 적용.** 위성의 *예측* pose를 따라가며 위치와 자세를 동시에 정렬하는 제어기(`coupled_predictive`)를 도입해 기본값으로 정하고, 기존 순차 제어기(`legacy`)도 설정으로 고를 수 있게 남겼습니다.

## 결과

| 시험 (시뮬레이션, Ground Truth 기준) | 결과 |
|---|---|
| 정지 위성 probe 도킹 | 20/20 성공 · 축 방향 오차 −9.98 mm, 반경 방향 6.53 mm |
| 표류 위성(0.02 m/s) 도킹 | 5회 중 3회 성공 |
| 6자유도 전체 임무 (포획 → 도킹 → 분리) | 끝까지 수행 |

> 이전 `legacy` 제어기로 개발 중 실행한 기록입니다. Isaac Sim에서 `legacy`와 `coupled_predictive`를 같은 조건으로 비교하지는 않았습니다.

## 한계

- 포획과 도킹 결합은 실제 래치나 흡착이 아닌 `FixedJoint` 근사입니다.
- 위성의 도킹 pose는 비전 추정이 아니라 시뮬레이터에서 가져옵니다.
- 이동 위성 도킹의 횡오차가 결합 한계 40 mm에 가깝게 남았습니다.

## 기술

Isaac Sim 5.0 · Isaac Lab · Space Robotics Bench · USD / PhysX · ROS 2 Jazzy · AprilTag · DLS IK · RGB-D 포인트 클라우드 · FastAPI · Firestore
