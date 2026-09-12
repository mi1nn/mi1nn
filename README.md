# Min (@mi1nn)

**기계공학으로 하드웨어를 배우고, 현장에서 AI 서버 장비를 설치 및 운영했고, 지금은 ROS2로 로봇을 움직입니다.**

---

## About

기계공학 학부에서 구조·진동 해석을 다루며 하드웨어를 이해했고, AI 헬스케어 스타트업에서 필드 엔지니어로 일하며 "잘 만든 시스템이 현장에서 어떻게 깨지는지"를 배웠습니다. 솔루션을 병원에 직접 설치하고 서버를 운영하면서, 제가 하고 싶은 일이 완성된 제품을 지원하는 쪽이 아니라 **하드웨어와 소프트웨어가 만나는 지점을 직접 설계하는 쪽**이라는 걸 알게 됐습니다. 그래서 로보틱스로 방향을 옮겼습니다.

현재 두산로보틱스 **ROKEY AI 부트캠프**에서 ROS2 기반 협동로봇 시스템을 개발하고 있습니다. 힘제어, 비전, 음성 인터페이스를 실제 로봇에 붙여보는 팀 프로젝트를 진행 중입니다.

- 🤖 관심 분야: 협동로봇 제어, Force/Torque 기반 정밀 조립, 로봇 비전 및 6D pose 추정
- 🛠 지금 공부하는 것: ROS2, MoveIt2 모션 플래닝, 쿼터니언 기반 pose 처리, RGB-D 카메라 영상 처리 및 YOLO 26s-seg 객체 탐지
- 📍 인천 근교 · 한국

---

## Projects

| 프로젝트 | 설명 | 핵심 기술 |
|---|---|---|
| **[assembly-cobot](https://github.com/mi1nn/assembly-cobot)** | F/T 센서 힘제어 기반 태양광 구조물 자동 조립 시스템. 야외 환경의 조명 변화·분진 때문에 비전을 쓸 수 없다는 제약에서 출발해, 3단계 힘제어 핀 삽입(1차 삽입 → 재파지 → 최종 삽입)으로 공차를 흡수하는 방식을 설계했습니다. MES 대시보드부터 로봇 제어 노드까지 전체 스택을 구현했습니다. | ROS2 · Doosan M0609 · Force Control · Flask · PostgreSQL |
| **[Project2-ROS2-VLA](https://github.com/mi1nn/Project2-ROS2-VLA)** | 음성 명령으로 동작하는 키트 조립 로봇. STT로 받은 자연어 명령을 키워드 추출·검증을 거쳐 로봇 작업 단위로 변환하고, RealSense 기반 객체 인식과 6D pose 추정으로 파지 위치를 결정합니다. | ROS2 · MoveIt · YOLO · Whisper · RealSense |
| **[mission_app](https://github.com/mi1nn/mission_app)** | 6축 로봇 모션 플래닝 실습 프로젝트. Move J/Move L 구분, pick-and-place 시퀀싱, `MultiThreadedExecutor`와 콜백 그룹 분리를 통한 데드락 회피 패턴을 다뤘습니다. | ROS2 · MoveIt2 · ros2_control |

<!-- mission_app이 private이면 이 행은 지우세요. 데모 영상이 있으면 각 설명 끝에 링크를 추가하면 효과가 큽니다. -->

---

## Tech Stack

**Robotics** ROS2 (Jazzy) · tf2 · Doosan Robotics API · ros2_control · OnRobot Gripper
**AI / Vision** YOLO · OpenCV · Intel RealSense · Whisper (STT) · PyTorch
**Backend** Python · PostgreSQL · Docker
**Engineering** CATIA · ANSYS (모달해석) · Six Sigma GB/BB · ISO 9001 / IATF 16949
**Tools** Git · Linux (Ubuntu) · Bash

---

## Background

- **두산로보틱스 ROKEY AI 부트캠프** — ROS2 협동로봇 개발 과정
- **AI 헬스케어 스타트업, CS/Field Engineer** — 솔루션 현장 배포, 서버 운영, 기술지원
- **K-Move 품질경영 과정 (620시간)** — ISO 9001/14001/45001, IATF 16949, APQP
- **기계공학 학사**
- **뉴질랜드 워킹홀리데이** — 영어 기반 고객 응대 경험

---

## Contact

- 📧 alekdi8gm30@gmail.com
- 📄 [이력서 / 포트폴리오](#)
