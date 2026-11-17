# Assembly-Cobot — 태양광 구조물 조립 협동로봇

**팀 프로젝트 (4명) · 두산로보틱스 ROKEY AI 부트캠프 · 2026.08**
📂 코드: [mi1nn/assembly-cobot](https://github.com/mi1nn/assembly-cobot) · 🇺🇸 [English](assembly-cobot.md) · ← [프로필](../README.ko.md)

<p align="center">
  <img src="https://github.com/mi1nn/assembly-cobot/raw/main/img/01_post.gif" width="49%" alt="기둥 삽입">
  <img src="https://github.com/mi1nn/assembly-cobot/raw/main/img/03_snapfit.gif" width="49%" alt="스냅핀 체결">
</p>

## 개요

두산 M0609 협동로봇이 기둥 → 프레임 → 스냅핀 → 패널 순서로 태양광 패널 구조물을 조립합니다. 야외의 조명 변화와 분진 때문에 비전을 쓰지 않고, F/T 센서 힘제어로 삽입 중 부품 공차를 흡수합니다.

## 담당

**PM, 시스템 아키텍처 설계**

## 한 일

- **프로젝트 리딩.** 팀과 범위를 정하고 요구사항, 테스트 계획, 최종 시나리오를 10개 요구사항 문서(비즈니스, 시스템, 기능, 인터페이스, 테스트, 하드웨어, DB, 운영, 유지보수)로 관리했습니다.
- **시스템 아키텍처 설계.** 웹 대시보드 → Flask 백엔드 → PostgreSQL → ROS 2 브리지 → 로봇 제어 노드로 이어지는 계층 구조를 설계했습니다. 로봇은 DB에 직접 쓰지 않고, 브리지만 둘 사이를 오갑니다.
- **실행 모델 설계.** 작업 지시 → 작업 실행 → 공정 실행의 3단계 구조로, 로봇의 모든 동작 단계가 DB 기록과 대응되게 했습니다. ROS 2 Action 목표 하나가 작업 실행 하나를 수행하고, 피드백으로 공정마다 상태를 갱신합니다.
- **로봇 쪽 작업.** 힘제어 루틴을 다듬고, 설치할 부품 정보를 코드에 고정하지 않고 DB에서 받아오도록 Action 인터페이스를 바꿨습니다.

## 힘제어 방식

- **2단계 삽입:** 1차 삽입 후 2차 가압으로 끝까지 밀어 넣습니다. 1차 삽입에 실패하면 0.3 mm 간격 25개 좌표에서 최적점을 찾아 다시 삽입합니다.
- **Periodic 끼워맞춤:** XY 방향으로 작게 진동시켜 프레임을 기둥에 끼우고 패널을 정렬합니다.
- **Arc 경로:** 2차함수 경로로 낮은 자재 위치에서 높은 설치 위치까지 접촉 없이 이동합니다.

## 기술

ROS 2 Jazzy · Doosan M0609 · DRL · F/T 힘제어 · Flask · SQLAlchemy · PostgreSQL
