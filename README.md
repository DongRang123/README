# VITAL LINK : AI기반 고령자 모니터링 시스템 

## 💡 서비스 개요

**VITAL LINK**는 고령화가 가속화된 사회에서 요양관리사의 부족해진 인력을 도와줄 수 있는 AI기반 고령자 모니터링 시스템입니다. 자율 주행 로봇 야간 순찰 및 시니어 분들의 실시간 건강 상태 감지를 기본으로 안전하고 지속 가능한 돌봄 환경을 제공합니다.


## 📝 주요 기능

- **실시간 생체 모니터링**
  - 웨어러블 아대를 통해 심박수, 산소포화도, 피부온도, 걸음 수, 낙상 여부 등을 수집
  - AI 기반 분석으로 위험 징후를 조기 감지 및 알림 전송

- **실시간 위치 추적**
  - BLE iBeacon 신호 세기(RSSI) 기반 구역(Location) 판별
  - 환자의 현재 위치를 실시간 서버와 연동하여 관리

- **응급 보고서 자동 생성**
  - 이상 징후 발생 시 AI 서버가 위험 상태를 탐지
  - MCP 서버를 통해 환자 메타데이터·최근 바이탈·인수인계 기록을 조합
  - AI 에이전트(Claude 3.5 Haiku)가 자동으로 마크다운 보고서 생성 및 PDF 저장

- **야간 순찰 자동화**
  - 자율주행 로봇이 LiDAR + SLAM을 활용하여 실내 경로 탐색 및 순찰 수행
  - YOLO 기반 Person 객체 탐지로 배회(Wander) 및 낙상(Fallen) 상황 인식
  - 탐지 시 이벤트 및 이미지 자동 전송


## 🔧 기술 스택

### 📱 HW (센서/디바이스)

- **사용자 센서**
  - BLE iBeacon 기반 위치 추적 (RSSI)
  - I2C 기반 바이탈 수집: 심박, 산소포화도, 체온, 걸음 수, 낙상 감지
  - SNTP 시간 동기화
  - MQTT 실시간 데이터 전송
    
- **환경 센서**
  - ESP32 BLE iBeacon Anchor (Major/Minor + RSSI)
  - 온도·습도·조도·TVOC 센서 (ADC / GPIO / I2C)
  - 1초 주기 센서 데이터 MQTT 전송 (JSON 형식)


### 🤖 Orin Car (자율주행 로봇)

- **프레임워크**
  - ROS2 Humble / SLAM Toolbox / RF2O Laser Odometry / Nav2
    
- **경로 탐색**
  - Hybrid A* (SmacPlannerHybrid), Regulated Pure Pursuit (RPP)
    
- **ROS 노드**
  - LiDAR, RF2O, 모터 드라이버, MQTT, 웨이포인트 매니저
  - 위치 데이터 MQTT 전송

- **객체 탐지**
  - YOLOv8n 기반 Person 탐지
  - Aspect ratio 기반 낙상 판정


### 🧠 AI 추론

- **데이터**
  - 바이탈: 심박, SpO₂, 체온, 걸음 수  
  - 환경: 온도, 습도, 조도, TVOC  
  - 메타: 나이, 성별, 기저질환 (하루 1회 갱신)  
  - 오픈 데이터셋: [AI-Hub 독거노인 위험감지](https://www.aihub.or.kr/aihubdata/data/view.do?currMenu=115&topMenu=100&dataSetSn=71803)

- **모델**
  - LSTM Autoencoder (시계열 + 메타 데이터 결합)
  - Lazy Loading (window=30, step=1)
  - ROC-AUC 기반 임계값 선정 → 이상 탐지

- **알림 & 보고서**
  - AI 서버 → MCP 서버 이상 이벤트 전송 (POST)
  - MCP 서버: MySQL(`patient`, `handover`), InfluxDB 조회  
  - Claude 3.5 Haiku → 마크다운 보고서 생성 → PDF 저장


### 🌐 WEB

- **Front-end**
  - Vue 3 / vue-router / Pinia
  - Axios (API 통신)
  - Chart.js + vue-chartjs (데이터 시각화)
  - Konva + vue-konva (2D 그래픽, 맵 오버레이)

- **Back-end**
  - FastAPI / Django REST Framework
  - MQTT (Mosquitto + Paho MQTT)
    - QoS 0: 초단위 센서 데이터
    - QoS 2: 로봇 제어 명령
  - SSE (Server-Sent Events) → 실시간 데이터 전송
  - DB
    - MySQL (환자 메타 정보)
    - InfluxDB (센서 시계열, 2일 보관)

- **시각화**
  - Grafana + InfluxDB → 실시간 대시보드


### ☁️ INFRA

- **CI/CD**: Jenkins (커스텀 빌드 이미지)

- **배포/호스팅**
  - Nginx (정적 리소스 + 리버스 프록시)
  - Docker (Grafana / Spring Boot / InfluxDB / Mosquitto 컨테이너)
  - Vercel (프론트엔드 호스팅)
  - AWS EC2 (서버 호스팅)


## 🔧 Project Setup

### 1. 환경 변수 설정 (.env)

프로젝트 루트 디렉토리에 `.env` 파일을 생성하고 아래 예시와 같이 환경변수를 입력하세요.

```env
# .env 예시
OPENAI_API_KEY=your_openai_api_key
ELASTIC_URL=https://your-elasticsearch-endpoint
ELASTIC_API_KEY=your_elastic_api_key
```

Backend
```sh
호
- **Position**: 
- **E-Mail**: 
- **username**: 

## 🧠 배운 점

- **실제 데이터 전처리와 통합의 중요성 체감**  
  여러 출처의 의약품·식품 데이터를 통합/정제하는 과정에서 데이터 불일치, 중복, 누락 문제를 직접 경험하며, 도메인 데이터 클린징과 스키마 표준화가 파이프라인 성능에 핵심이라는 점을 배움.

- **벡터DB/검색 인덱스 활용 역량 강화**  
  Elasticsearch를 활용해 키워드 검색과 벡터 검색을 동시에 구현, 오타/유사어 처리(Fuzzy Search) 기법을 실전에서 익힘.

- **GPT-4o-mini 등 LLM 연동 및 RAG 구조 설계**  
  LLM 단독 답변의 한계(할루시네이션 등)를 RAG로 극복하는 구조를 경험, 프롬프트 설계와 context 제공이 응답 품질에 큰 영향을 미침을 체감.

- **실제 챗봇 서비스 배포 및 운영 노하우 습득**  
  AWS, Vercel 등 다양한 인프라 환경에 배포·운영하며, 클라우드 서비스와 서버리스 환경의 장단점을 직접 체험.

- **RAGAS 기반 자동 평가와 정성 평가 병행의 필요성**  
  단순 자동화된 정량 평가 외에, 실제 사용자 입장에서의 정성 평가(질문 샘플링, 응답의 이해도·관련성 평가)가 챗봇 완성도에 반드시 필요함을 느낌.

- **프로젝트 협업의 커뮤니케이션 경험**  
  데이터팀, 프론트엔드, 백엔드 등 여러 역할 간 협업 과정에서 API 설계와 데이터 흐름 문서화의 중요성을 실감.
