# Min (@mi1nn)

**ROS 2 기반 로봇 엔지니어 — 협동로봇 힘제어 · AMR 자율주행 · 시뮬레이션 검증**

🇺🇸 [English](README.md)

---

## About

기계공학을 전공하며 구조·진동 해석을 다뤘습니다. 이후 AI 헬스케어 스타트업에서 필드 엔지니어로 일하며 병원에 솔루션을 설치하고 그 뒤의 서버를 운영했습니다. 그 일을 하며 배포된 AI 시스템이 실제 현장에서 어떻게 돌아가고 어디서 문제가 생기는지를 배웠습니다. 동시에 다음에 하고 싶은 일도 분명해졌습니다. 완성된 제품을 지원하는 것이 아니라, **하드웨어와 소프트웨어가 만나는 지점을 직접 설계하는 일**입니다. 그래서 로보틱스로 방향을 옮겼습니다.

두산로보틱스 **ROKEY AI 부트캠프**에서 ROS 2 기반 로봇 시스템을 개발했습니다. 실제 협동로봇의 힘제어 조립, 웹캠과 AMR이 협업해 움직이는 대상을 추적하는 시스템, Isaac Sim에서 시뮬레이션한 궤도 서비싱 임무를 팀 프로젝트로 진행했습니다.

- 🤖 관심 분야: 협동로봇 제어, Force/Torque 기반 정밀 조립, 로봇 비전 및 6D pose 추정, 시뮬레이션 기반 로봇 검증
- 🛠 지금 공부하는 것: C++, SLAM, MoveIt 2 모션 플래닝
- 📍 인천 · 한국

---

## Projects

| 프로젝트 | 설명 | 담당 | 핵심 기술 |
|---|---|---|---|
| **[satellite-mission-extension-system](https://github.com/mi1nn/satellite-mission-extension-system)** | Isaac Sim에서 궤도 위성 서비싱 임무를 시뮬레이션합니다. 7자유도 로봇팔이 AprilTag로 표류하는 모듈을 추적·포획해 이동 중인 위성에 도킹합니다. Astrobee가 포인트 클라우드로 도킹 통로를 확인해 장애물이 있으면 도킹을 중단합니다. | 환경 세팅, 도킹 구현 | Isaac Sim · Isaac Lab · ROS 2 · AprilTag · Point Cloud · FastAPI · Firestore |
| **[overwatch-amr](https://github.com/mi1nn/overwatch-amr)** | 움직이는 RC카를 찾아 따라가는 TurtleBot 4 추적 시스템입니다. 고정 웹캠이 YOLO로 RC카 위치를 알려주면 로봇이 Nav2로 접근하고 OAK-D depth로 추적합니다. 대상을 놓치면 상태 머신이 회전 탐색으로 다시 찾습니다. | SLAM 지도 작성, OAK-D 카메라용 YOLO 학습 | ROS 2 · Nav2 · AMCL · SLAM · TurtleBot 4 · OAK-D · YOLO26 |
| **[assembly-cobot](https://github.com/mi1nn/assembly-cobot)** | 두산 협동로봇으로 태양광 구조물을 자동 조립합니다. 야외의 조명·분진 때문에 비전 대신 3단계 힘제어 삽입으로 부품 공차를 흡수합니다. MES 대시보드에서 로봇 제어까지 작업 흐름 전체를 관리합니다. | PM, 시스템 아키텍처 설계 | ROS 2 · Doosan M0609 · Force Control · Flask · PostgreSQL |
| **[Project2-ROS2-VLA](https://github.com/mi1nn/Project2-ROS2-VLA)** | 음성 명령으로 동작하는 키트 조립 로봇입니다. STT로 받은 명령을 검증을 거쳐 로봇 작업으로 변환합니다. 객체 탐지와 6D pose 추정으로 파지 위치를 결정합니다. | 비전 (객체 탐지, OctoMap) | ROS 2 · MoveIt · YOLO · OctoMap · Whisper · RealSense |

---

## Tech Stack

**Languages** Python · C++ 

**Robotics** ROS 2 (Jazzy) · Nav2 · AMCL · SLAM · tf2 · ros2_control

**Simulation** NVIDIA Isaac Sim · Isaac Lab · USD / PhysX

**AI / Vision** YOLO · OpenCV · AprilTag · OAK-D · RGB-D Point Cloud · OctoMap · PyTorch

**Engineering** CATIA · ANSYS (모달해석) · Six Sigma GB/BB

**Tools** Git · Docker · PostgreSQL · Linux (Ubuntu) · Bash

---

## Background

- **ROKEY 부트캠프 교육센터, 조교 인턴** — 2026.11 ~
- **두산로보틱스 ROKEY AI 부트캠프** — ROS 2 협동로봇·AMR 개발 과정 수료
- **AI 헬스케어 스타트업, CS/Field Engineer** — 솔루션 현장 배포, 서버 운영, 기술지원
- **K-Move 미래 모빌리티 품질경영 과정 (620시간)** — ISO 9001/14001/45001, IATF 16949, 비즈니스 영어 회화
- **기계공학 학사** — 4대 역학(동역학, 열역학, 유체역학, 고체역학), 기구학, 수치해석, 기계진동
- **뉴질랜드 워킹홀리데이** — 영어 기반 고객 응대 경험

---

## Contact

- 📧 alekdi8gm30@gmail.com
