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
- **레퍼런스 기반 도킹 제어기 적용.** 위성의 *예측* pose를 따라가며 위치와 자세를 동시에 정렬하는 제어기(`coupled_predictive`)를 도입해 기본값으로 정하고, 기존 순차 제어기(`legacy`)도 설정으로 고를 수 있게 남겼습니다. 자세한 내용은 [아래](#레퍼런스-기반-도킹-제어기)에 있습니다.

## 레퍼런스 기반 도킹 제어기

### 왜 필요했나
로봇팔이 3,000 kg짜리 MEP를 들고 있어서, 팔과 화물이 주기 약 20초로 천천히 흔들리고 감쇠도 거의 없었습니다. 기존 `legacy` 제어기는 이 흔들림을 건드리지 않으려고 아주 느리게 움직이고, 위치를 먼저 맞춘 뒤 자세를 맞추는 순차 방식이었습니다. 그 결과 표류하는 위성에 도킹할 때 횡오차가 39.97–39.98 mm로, 결합 한계 40 mm에 바짝 붙어 있었습니다.

### 적용한 논문

| 레퍼런스 | 가져온 개념 | 적용 방법 |
|---|---|---|
| Zhou, Liu, Cai — *Motion-planning and pose-tracking based rendezvous and docking with a tumbling target* | "도킹점이 어디에 있을지"와 "어떻게 따라갈지"를 분리 | 위성의 선속도·각속도로 0.3초 뒤 도킹점 pose를 예측(constant-twist 모델)하고, 예측된 pose를 추종 |
| Ye, Lu, Mu — *Compound control for autonomous docking to a three-axis tumbling target* | 위치·자세·상대 선속도/각속도를 함께 추종 | 위치와 자세를 동시에 맞추는 feed-forward + P + D 추종기. 자세 오차는 SO(3) 회전 벡터 $e_R=\log(R_dR_c^T)$로 계산 |
| Chen & Sun (2020) — *Nonlinear control of underactuated systems subject to both actuated and unactuated state constraints* | 켜고 끄는 방식 대신 연속적인 제약 | 정렬이 어긋난 정도(10–80 mm, 1–7°)에 비례해 접근을 늦추는 soft gate와, 복귀 조건의 히스테리시스 |
| Singer & Seering (1990) — Input Shaping | 계단식 명령은 잔류 진동을 일으킴 | 속도를 계단식으로 바꾸지 않고 완만하게 증가 |

**예측과 추종**

$$p_d(t+T)=c+v_cT+\exp([\omega]T)\,(p_d(t)-c)$$
$$v_{cmd}=v_D+\operatorname{clip}(K_p e_p)-K_d(v_{tip}-v_D)$$

**정렬 확인:** 횡오차 50 mm 미만, 자세 오차 4° 미만, 상대 속도 0.02 m/s 미만, 상대 회전 1°/s 미만이 0.5초 동안 연속으로 성립해야 삽입을 시작합니다. 150 mm나 12°를 넘으면 비상 정지합니다.

### 검토했지만 적용하지 않은 것
- **슬라이딩 모드 제어**(Ye 외): 스위칭이 20초 흔들림 모드를 자극할 수 있었습니다.
- **중량 화물 임피던스 제어**(Aghili, 2009): 게인 1.0의 능동 감쇠가 0.1초 만에 발산했고, 안정한 게인을 찾지 못했습니다.
- **Reaction Null-Space**(Nenchev 외, 1999)와 **다중 모드 Input Shaping**(Singhose, 2009): 베이스가 고정돼 있고 흔들림 모드가 하나뿐이라 실제 문제와 맞지 않았습니다.
- **Flatness 기반 궤적 계획:** 궤적을 미리 계산하는 방식이라, 매 스텝 표류하는 목표를 다시 읽는 폐루프 도킹과 맞지 않았습니다.

### 튜닝: 정렬이 너무 느린 문제
새 제어기로 처음 Isaac Sim을 돌렸을 때, 자세는 금방 맞았지만 횡오차가 초당 약 7 mm씩만 줄어 정렬에 50초가 더 걸릴 상황이었습니다. 제어 식을 보면 오차가 클 때 P 항은 상한에 걸리고 D 항이 그 움직임만큼 다시 빼기 때문에, 속도가 **상한 ÷ (1 + K_d)** = 15 ÷ 1.6 ≈ 9.4 mm/s로 수렴합니다. toy 모델(8.0 mm/s)과 시뮬레이터(약 7 mm/s) 값과도 맞았습니다. 오프라인 toy 모델로 상한 15–30 mm/s를 비교해, 상한을 0.03 m/s로 올렸을 때 정렬 시간이 41초에서 27초로 줄고 오버슈트도 가장 작은 것을 확인했습니다.

### 검증 상태
- 오프라인 단위 테스트로 예측, SO(3) 오차, gate, 히스테리시스, 속도 추정을 검증했습니다.
- 41초 → 27초는 오프라인 toy 모델 결과입니다. **Isaac Sim에서 `legacy`와 `coupled_predictive`를 같은 조건으로 비교하는 시험은 마치지 못해서**, 도킹 성공률이나 오차 개선은 주장하지 않습니다.

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
