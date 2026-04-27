# ROBO Katsu

**Doosan Robotics M0609** 협동 로봇을 활용하여 밥 푸기, 반찬 배식, 소스 과정을 자동화한 ROS2 기반 제어 프로젝트

---

## 1. 운영체제 환경 (Environment)
* **OS:** Ubuntu 22.04 LTS
* **ROS 2:** Humble 
* **Language:** Python, DLR

---

## 2. 사용 장비 목록 (Hardware)
* **Robot:** Doosan Robotics M0609 
* **Computing:** Lenovo Legion 5 (Ryzen 7 260, RTX 5060)
* **End-Effector:** GripperDA_v1 (TCP)

---

## 3. 시스템 설계 및 플로우 차트 (System Design)

### 3.1 주요 제어 로직
본 시스템은 각 음식의 특성에 따라 세 가지 핵심 제어 방식을 사용한다.

1. **힘 제어 기반 밥 푸기 (Force Control):** `task_compliance_ctrl`을 통해 Z축 바닥 접촉을 감지(`check_force_condition`)하고, 반원 궤적(`movesx`)을 그려 밥을 퍼올린다.
2. **비동기 주기적 모션 (Periodic Motion):** `amove_periodic`을 사용하여 로봇이 좌우로 흔들리는 동안 그리퍼(`set_digital_output`)를 조여 소스를 짜는 동작을 동시에 수행한다.
3. **디지털 I/O 그리퍼 제어:** 디지털 출력값 조합을 통해 그리퍼의 상태를 3단계(**RELEASE / BASIC / TIGHT**)로 정밀 제어한다.

### 3.2 프로세스 플로우 및 아키텍처
<img width="4308" height="2464" alt="image" src="[https://github.com/user-attachments/assets/5de331ad-278e-4f11-8d00-74f6b33af30c](https://github.com/user-attachments/assets/5de331ad-278e-4f11-8d00-74f6b33af30c)" />

본 프로젝트는 시스템의 부하를 분산하고 조작 편의성을 높이기 위해, 로봇 직접 제어부(Device 1)와 원격 명령/UI 관리부(Device 2)로 나뉜 **멀티 디바이스 분산 아키텍처**를 채택했다. 

#### 📍 주요 구성 요소 (System Components)

**1. Device 1: 로봇 직접 제어 및 상태 퍼블리셔 (Robot Control PC)**
실제 Doosan M0609 로봇과 통신하며 물리적인 동작을 수행하는 핵심 장비이다.
* **Robot Status Publisher:** 로봇의 실시간 상태(위치, 조인트 각도, 힘 등)를 읽어와 ROS2 `Topic`(분홍색 선)으로 발행한다.
* **Controller Node & DSR Node:** Device 2로부터 모션 제어 명령을 `Service`(파란색 선) 형태로 수신하여, 내부 태스크 스레드를 통해 `DSR2 Controller`에 로봇 구동 명령을 하달한다.

**2. Device 2: 원격 태스크 관리 및 Web UI (Task & UI PC)**
사용자 입력을 처리하고 전체적인 작업 프로세스(Task)를 조율하는 외부 장비이다.
* **Task Controller:** 사용자의 명령(Web UI)을 받아 Device 1으로 적절한 `Service` 요청을 보내며, Device 1에서 퍼블리시하는 상태 `Topic`을 구독하여 모션을 모니터링한다.
* **UI Bridge (rosbridge websocket):** ROS2 통신 환경과 로컬 웹 환경을 연결하는 브리지 노드로, 웹 브라우저 기반의 조작을 가능하게 한다.
* **Web UI (Localhost):** 관리자가 로봇 시스템을 직관적으로 제어하고 상태를 확인할 수 있는 인터페이스이다.

**3. 클라우드 연동 (Cloud Integration)**
* **Firebase:** 시스템의 주요 상태(Status)와 이벤트(Event) 로그는 Firebase로 전송되어, 원격 모니터링 및 데이터 관리를 지원한다.

#### 🔄 주요 통신 방식 (ROS2 Communication)
설계도에 명시된 바와 같이 두 가지 핵심 ROS2 통신 프로토콜을 사용한다.
* 🟦 **Service (Blue):** 확실한 응답이 필요한 로봇 모션 트리거 및 명령 제어 (Device 2 $\rightarrow$ Device 1)
* 🟪 **Topic (Pink):** 끊김 없이 스트리밍되어야 하는 로봇 상태 데이터 구독 (Device 1 $\rightarrow$ Device 2)

---

## 4. 사용 설명 (Usage Guide)

본 시스템은 두산 로봇 드라이버를 통해 실제 로봇과 연결된 상태에서 작동한다. 원활한 실행을 위해 두 개의 터미널을 열어 순차적으로 명령어를 입력한다.

### 실행 순서 (Execution Steps)

#### Step 1: 로봇 드라이버 실행 및 실로봇 연결
첫 번째 터미널에서 아래 명령어를 입력하여 실제 **M0609** 로봇과 통신을 시작하고 RViz 시각화 툴을 실행한다.

```bash
ros2 launch dsr_bringup2 dsr_bringup2_rviz.launch.py mode:=real host:=192.168.1.100 port:=12345 model:=m0609
```
#### Step 2: 로봇 제어 노드 실행
```bash
ros2 run cobot1 파이썬 파일
```
