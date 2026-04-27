# Doosan Robotics M0609 자동 배식 시스템 (ROS2)

**Doosan Robotics M0609** 협동 로봇을 활용하여 밥 푸기, 반찬 배식, 소스 도포 과정을 자동화한 ROS2 기반 제어 프로젝트입니다.

---

## 1. 운영체제 환경 (Environment)
* **OS:** Ubuntu 22.04 LTS
* **ROS 2:** Humble Hawksbill
* **Language:** Python 3.10
* **Robot Interface:** `DSR_ROBOT2` (rclpy 기반)

---

## 2. 사용 장비 목록 (Hardware)
* **Robot:** Doosan Robotics M0609 (Collaborative Robot)
* **Computing:** * Lenovo Legion 5 (Ryzen 7 260, RTX 5060)
    * Desktop (Ryzen 7 5700G, Gigabyte B450M AORUS Elite)
* **Network:** TP-Link Archer T2U Nano (Wireless Connect)
* **End-Effector:** GripperDA_v1 (TCP 설정명)

---

## 3. 시스템 설계 (System Design)

### 3.1 주요 제어 로직
본 시스템은 각 음식의 특성에 따라 세 가지 핵심 제어 방식을 사용합니다.

1. **힘 제어 기반 밥 푸기 (Force Control):** - `task_compliance_ctrl`을 통해 Z축 바닥 접촉을 감지(`check_force_condition`)하고, 반원 궤적(`movesx`)을 그려 밥을 퍼올립니다.
2. **비동기 주기적 모션 (Periodic Motion):** - `amove_periodic`을 사용하여 로봇이 좌우로 흔들리는 동안 그리퍼(`set_digital_output`)를 조여 소스를 짜는 동작을 동시에 수행합니다.
3. **디지털 I/O 그리퍼 제어:**
   - 디지털 출력값 조합을 통해 그리퍼의 상태를 3단계(**RELEASE / BASIC / TIGHT**)로 정밀 제어합니다.
## 1. 시스템 설계 및 플로우 차트 (System Design)

### 1.1 프로세스 플로우
![Process Flowchart](./image.png)
> 밥 푸기(힘 제어), 소스 도포(주기적 모션), 배식(좌표 제어)의 단계별 흐름도입니다.

---

## 5. 사용 설명 (Usage Guide)

본 시스템은 두산 로봇 드라이버를 통해 실제 로봇과 연결된 상태에서 작동합니다. 실행을 위해 두 개의 터미널이 필요합니다.

### 실행 순서 (Execution Steps)

#### Step 1: 로봇 드라이버 실행 및 실로봇 연결
첫 번째 터미널에서 아래 명령어를 입력하여 실제 **M0609** 로봇과 통신을 시작하고 RViz 시각화 툴을 실행합니다.
```bash
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=real host:=192.168.1.100 port:=12345 model:=m0609
