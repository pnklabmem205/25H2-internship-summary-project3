---

## 1. 개요

### 👥 프로젝트 정보

- 진행자: 
- 기간: 2025년 11월 24일 ~ 2025년 12월 12일
- 프로젝트 목표: **907호 출입자 실시간 얼굴 인증 기반 스마트 출입 통제 시스템을 구현**
    - ESP32-S3-CAM 단독으로 **얼굴 검출 → 얼굴 인식 → 인증 성공/실패 판단** 수행
    - 인증이 확정되면 서버에 출입 로그를 저장해 **출입 이력 자동 기록**
    - 카메라를 서보모터(PAN/TILT) 위에 올려 **얼굴 방향으로 카메라가 움직이는 PTZ 기능**까지 구현
        - https://www.youtube.com/watch?v=f2TUxoaKIsA
- 깃허브: https://github.com/pnkmem432/esp-who-face_entry— 보드 / https://github.com/pnkmem432/esp-who-face_entry_server — 서
- 구글 드라이브: https://drive.google.com/drive/folders/1FFd2yBBWBz2TWsMGH4dfdfB4-6MjLH9F?hl=ko

---

### 💡 프로젝트 요약

**본 프로젝트는 907호 출입을 보다 체계적으로 관리하기 위해, ESP32-S3-CAM 단독으로 얼굴을 검출·인식하고 인증 결과를 서버에 출입 로그로 남기는 IoT 출입 인증 시스템을 구축하였다.**

초기에는 ESP-WHO 예제를 기반으로 스트리밍과 얼굴 검출을 동시에 붙이는 과정에서 프레임 포맷(JPEG/RGB656) 불일치와 카메라 프레임을 여러 곳에서 동시에 읽는 구조로 인해 스트리밍 실패 문제가 발생했지만, 이후 단일 캡처 테스크 구조로 안정화하고 얼굴 박스 오버레이까지 구현하였다.

ESP-WHO 정식 얼굴 임베딩 모델을 사용해 이름+유사도(%) 인식이 가능해졌고, SPI Flash 영구 저장, 사용자 등록/목록/삭제 기능을 추가해 운영 가능한 형태로 확장하였다. 이어서 인증 흐름을 상태머신으로 안정화하고 LED로 결과를 표시하여 사용자의 출입 인증 성공 유무를 직관적으로 알 수 있도록 하였다.

이후에는 인증 성공 순간 서버로 출입 로그를 POST 하여 DB에 저장하고, 운영 편의를 위한 Threshold 슬라이더, 최근 출입자 표시 등의 기능을 추가함으로써 완성도를 높였다. 그리고 서보모터 기반 PTZ 얼굴 트래킹까지 구현하여 데모 가능한 프로토타입 수준으로 완성하였다. 

---

### 🎯 목표 및 기대효과

**구체적 목표**

- ESP32-S3-CAM에서 **얼굴 인증을 로컬에서 완결**하여 출입 여부를 결정
- 인증 결과를 서버 DB에 저장해 **출입 로그 자동 관리** 구현
- 관리자 UI로 등록/삭제/목록/로그 조회를 가능하게 해 **운영 가능한 시스템** 구축
- 카메라 PTZ(서보) 기능으로 **사용자 얼굴을 화면 중앙에 위치**시키는 보조 기능 구현

**기대효과**

- 출입 기록의 관리 효율 상승
- 로컬 인증 구조로 네트워크/서버 부하 최소화
- PTZ 기능으로 얼굴 인식 성공률 향상 기대

## 2. 시스템 및 기술 구성

### 2.1 전체 구성

### ⚙️ ESP32-S3-CAM

- 카메라 프레임 캡쳐
- 얼굴 검출
- 얼굴 임베딩 생성
- 로컬 얼굴 DB 매칭
- 인증 상태머신 + LED 표시
- 인증 성공 시 서버로 출입 로그 POST
- PTZ 얼굴 트래킹

### ⚙️ 서버 (FastAPI + PostgreSQL)

- 출입 로그 저장/조회 API 제공
- 얼굴 DB 백업 저장/복구 API 제공

### ⚙️ 관리자 UI (ESP32 내장 웹 페이지)

- /스트리밍
- /사용자 등록
- /사용자 목록
- /사용자 삭제
- /출입 로그

### 2.2 개발 환경/버전

### **ESP-IDF**

- Espressif Systems가 제공하는 ESP32 마이크로컨트롤러를 위한 공식 IoT 개발 프레임워크
- 다운로드: https://dl.espressif.com/dl/esp-idf/?utm_source=chatgpt.com or https://github.com/espressif/esp-idf.git
- 버전: ESP-IDF v5.4.2

### **ESP-WHO**

- Espressif 사에서 개발한 ESP32 칩을 기반으로 얼굴 인식 및 기타 머신러닝 작업을 쉽게 구현할 수 있도록 만든 프레임워크
    
    ![image.png](attachment:75f6900b-4c00-439d-957d-956135f2e97d:image.png)
    
    - **human_face_recognition**
        - 얼굴 검출 (Face Detect)
        - 얼굴 특징 추출 (Embedding)
        - 얼굴 인식 (Recognize)
        
        → ESP-WHO의 얼굴 검출, 특징 추출, 인식은 모두 딥러닝(CNN) 기반이고, 학습이 끝난 모델을 가져다 쓰는 방식이므로 ESP32에서 학습은 하지 않고 추론만 진행함
        
    - object_detect
        - 물체 탐지
    - qrcode_recognition
        - QR 코드 위치 탐색
        - QR 디코딩 (문자열 추출)
- 다운로드: https://github.com/espressif/esp-who
- 버전: v0.9.4-258-g9f6d69d

### **ESP32-S3-CAM (OV2640)**

[esp32-s3_datasheet.pdf](attachment:99191895-f3f5-4545-8d91-92ba654635f2:esp32-s3_datasheet.pdf)

- 특징
    - ESP32-S3는 기존 ESP32보다 AI/영상 처리에 특화된 MCU
        - *Xtensa LX7 Dual-core CPU (최대 240MHz)*
        - *Vector instructions (SIMD) 지원*
            - 얼굴 감지/인식은 사진 → 숫자 → 수학 연산으로 처리됨
            - 일반 MCU는 이 연산들이 느리지만,
                
                ESP32-S3는 AI 계산을 빠르게 하기 위한 전용 명령어(SIMD) 를 포함하고 있음
                
            - 그래서 TinyML(TensorFlow Lite Micro) 모델을 MCU 내부에서 빠르게 돌릴 수 있음
    - 카메라 연결 전용 하드웨어(DVP 인터페이스) 내장
        - DVP(Digital Video Port)
            - 카메라 센서가 MCU에게 픽셀 데이터를 디지털 신호로 보내는 방식
        - *Parallel 8-bit Camera Interface 지원*
            - 카메라를 위한 전용 핀 + 전용 회로가 칩 내부에 내장되어 있음
            - OV2640 / OV5640
            
            같은 DVP(Camera Parallel Interface)를 사용하는 카메라들을
            
            그대로 꽂으면 바로 영상 데이터를 읽을 수 있음
            
    - 내장 Wi-Fi + AP 모드 지원
        - *802.11 b/g/n Wi-Fi Subsystem*
        - *SoftAP 모드(Access Point Mode) 지원*
            - 공유기에 연결되는 STA 모드 (ESP32가 공유기에 접속하는 모드)
            - 보드 자체가 Wi-Fi를 발사하는 AP 모드 (ESP32가 공유기 역할을 하는 모드)
            - 즉, ESP32가 작은 Wi-Fi 공유기가 될 수 있음
    - USB-C 하나로 전원/업로드/시리얼 출력 모두 가능
        - *USB 1.1 OTG 기능 내장*
        - *USB-CDC(가상 시리얼) 지원*
            - ESP32-S3는 USB 기능이 칩 안에 직접 들어있어 전원 공급, 펌웨어 업로드, Serial Monitor 출력 모두 가능
- CAM
    
    
    ![image.png](attachment:fad94f2b-c3a3-40bf-b572-28530698eca9:image.png)
    
    - **OV2640**
    
    ![image.png](attachment:950533af-e07b-4548-b8d1-fcd63933a877:image.png)
    
    - **OV5640**
    
    ### **기본 스펙 비교**
    
    | 항목 | **OV2640** | **OV5640** |
    | --- | --- | --- |
    | **해상도** | 2MP (UXGA 1600×1200) | 5MP (2592×1944) |
    | **픽셀 크기** | 2.2 μm × 2.2 μm | 1.4 μm × 1.4 μm |
    | **프레임레이트** | UXGA 15fps, SVGA 30fps, CIF 60fps | 1080p 30fps 지원 |
    | **출력 포맷** | JPEG, YUV422/420, RGB565/555, Raw8/10 | JPEG, YUV, RGB, Raw10/8 |
    | **렌즈 사이즈** | 1/4" | 1/4" |
    | **S/N Ratio** | 40 dB | 약 38 dB |
    | **인터페이스** | DVP (8bit/10bit Parallel) | DVP + MIPI (2Lane) |
    | **전력 소비** | 125mW (UXGA 15fps) | 더 높음(고해상도라 전력 ↑) |
    - **해상도: 사진이 얼마나 선명한지**
        - 숫자가 클수록 더 많은 점(픽셀)로 이미지를 만들기 때문에 얼굴이 더 선명하게 보임
        - 고해상도 일수록 얼굴 디테일이 많아서 인식 정확도가 증가할 것으로 예상
        
        ![image.png](attachment:5b6f8848-2f66-4cb6-88cf-40ba0973ab11:image.png)
        
        - 영상 해상도 종류
            - 카메라가 내보내는 화면 크기(가로x세로 픽셀 수)를 말하는 등급
                - 사진/영상 크기가 종류 별로 FHD, 4K, 8K 이런 이름이 있는 거 처럼
                카메라 센서는 아래와 같이 분류
            1. UXGA(1600x1200) 
                - **U**ltra e**X**tended **G**raphics **A**rray
                - 1600 × 1200 ≈ 192만 픽셀 (약 2MP)
                - OV2640의 최고 해상도
                - fps(초당 프레임수)가 떨어짐 → **15fps**
                - 고해상도 사진 수준, 영상은 조금 끊길 수 있음
            2. SVGA(800x600)
                - **S**uper **V**ideo **G**raphics **A**rray
                - 800 × 600 = **480,000 픽셀 (0.48MP)**
                - 해상도는 낮지만 fps가 올라감 → 30fps
                - 즉, UXGA보단 흐리지만 영상은 조금 더 부드러울 수 있음
            3. CIF(352x288)
                - **C**ommon **I**ntermediate **F**ormat
                - 352 × 288 = **약 100,000 픽셀**
                - 매우 낮은 해상도
                - 대신 **fps가 매우 높음 → 60fps 가능**
                    - Tracking, 빠른 움직임 감지에 유리
                - 즉, 화질은 매우 낮지만 엄청 부드러운 영상일 수 있음
            4. 1080p(1920x1080)
                - Full HD (FHD)의 영상 규격
                - 1920 × 1080 = **약 2.07MP**
                - 고화질
                - fps는 보통 30fps
                - 유튜브 Full HD 화질과 같음
    - **픽셀 크기: 빛을 받는 작은 센서 점의 크기**
        - 픽셀이 크면 → 한 픽셀에 더 많은 빛이 들어옴 (어두운 곳에서 유리)
        - 픽셀이 작으면 → 고해상도 구현 가능 (선명한 사진 가능)
    - **프레임레이트(FPS): 1초에 몇 장의 사진을 보여줄 수 있는지**
        - 30fps = 1초에 30장 → 자연스러운 영상
        - 15fps = 약간 끊김
        - 60fps = 매우 부드러움
        - 얼굴 Tracking(서보 따라가기) 하려면 fps 중요할 수 있음
    - **출력 포맷: 카메라가 찍은 영상을 어떤 형태로 내보내는지**
        - JPEG: 우리가 아는 일반 사진 파일
        - YUV: 영상용 색 표현 방식
        - RGB565: 빨/초/파 를 숫자로 저장
        - Raw8/10: 보정 전 raw 데이터
    - **렌즈 사이즈: 카메라 렌즈 크기**
    - **S/N Ratio(Signal to Noise Ratio): 사진에 노이즈(지글지글한 점)이 얼마나 적은지**
        - 숫자가 높을수록 깨끗한 이미지
        - 낮을수록 노이즈가 많음
    - **인터페이스(DVP,MIPI): 카메라와 MCU가 어떻게 대화하느냐(데이터 전송 방식)의 차이**
        - DVP: ESP32 가 지원하는 일반적인 8비트 병렬 방식
        - MIPI: 더 빠른 직렬 방식(스마트폰 카메라에서 사용)
    
    ---
    
    ### **영상 처리 기능 비교**
    
    | 기능 | OV2640 | OV5640 |
    | --- | --- | --- |
    | 자동 노출(AEC) | 지원 | 고급 AEC (안정적) |
    | 자동 화이트밸런스(AWB) | 지원 | 더 정확한 AWB |
    | 자동 게인(AGC) | 지원 | 성능 더 좋음 |
    | 렌즈 보정 | 지원 | 더 고급 보정 |
    | JPEG 엔코더 | 내장 (빠름) | 더 고화질 JPEG |
    | 노이즈 감소 | 기본 | 고급 NR 처리 |
    
    →  OV5640 = **센서 자체 영상 처리 품질이 훨씬 좋음**
    
    ![image.png](attachment:ef1b793b-cdde-4636-915f-9ddc29e91c0e:045a24e1-05c0-4536-ba28-87427cd2f9da.png)
    
    https://www.youtube.com/watch?v=BCvOBMQSliY
    
    ### **1단계: OV2640으로 초기 프로토타입 개발**
    
    이유:
    
    1. ESP32-CAM, ESP32-S3-CAM 대부분이 **기본 내장 카메라로 OV2640**을 사용
        - 관련 자료가 많음
    2. OV2640은 메모리 부담이 적어서 **초기 세팅이 훨씬 안정적이고** 빠른 테스트/디버깅 가능
        - OV2640은 **압축된 작은 JPEG 덩어리**만 ESP32에 전달
        - OV5640 같은 고성능 센서는 **압축을 덜 하고 RAW 데이터나 큰 데이터 블록**을 보냄
    3. TinyML 모델은 입력으로 **저해상도 이미지(예: 96×96, 160×160)**를 사용
        - 얼굴 감지 시 딜레이가 적기 때문에 얼굴만 잘 찾는 기능을 개발하기에는 최적
    
    ### **2단계: OV5640 기반 정확도 향상 확장**
    
    이유:
    
    1. **해상도 2MP → 5MP로 증가**
        - 얼굴 디테일 증가 → 얼굴 인식 정확도 향상
    2. **최종 완성도 향상**
        - 등록된 사용자 수 증가 시 정확도 유지에 유리

![image.png](attachment:307091b0-6356-4239-a3dc-939f5a0a8318:image.png)

### 2.3 주요 구성 요소 역할

### 2.3.1 ESP32-S3 얼굴 인식 단말기

![image.png](attachment:376da0de-811c-4102-86df-63ea0d9d0b64:image.png)

- OV2640 카메라 사용
- 실시간 얼굴 인식 Loop 수행
    
    **1) 얼굴 검출(Detection)**
    
    - 입력: RGB 이미지
    - 출력: 얼굴 박스 좌표(x1,y1,x2,y2), score
    - 목적: 얼굴이 어디에 있는지 찾는 단계
    
    **2) 얼굴 임베딩(Embedding)**
    
    - 얼굴 영역을 **512차원 벡터**로 변환
    - 이 벡터는 얼굴의 특징을 숫자로 요약한 값
    - 같은 사람은 벡터가 비슷하게, 다른 사람은 멀게 나오도록 학습되어 있음
    
    **3) 얼굴 인식(Recognition)**
    
    - 현재 얼굴 임베딩 vs 저장된 임베딩을 비교
    - 비교 방법: 코사인 유사도(cosine similarity)
    - 유사도가 threshold 이상이면 등록자로 인정
        - threshold는 코드 상 설정할 수 있음
        - 이후 코드로 수정하는 것이 아닌 사용자가 직접 설정할 수 있도록 UI 구현
    
    **4) 인증 상태머신(Auth State Machine)**
    
    - 단순히 한 프레임 결과로 SUCCESS/FAIL을 내면 흔들릴 수 있음
    - IDLE→CANDIDATE→AUTHENTICATING→SUCCESS/FAIL→COOLDOWN 의 상태머신으로 안정화
    
    **5) 서버 로그 저장**
    
    - 인증 성공 확정 순간에만 1회 POST
    
    **6) PTZ(서보) 트래킹**
    
    - 얼굴 박스 중심이 화면 중앙에서 벗어난 정도를 이용해 PAN/TILT 각도를 조금씩 조정하여 얼굴을 중앙에 두도록 함
    
- 출입 판단 후 서버로 POST
- Web UI를 위한 HTTP 동작 
(실시간 스트리밍 + 등록/삭제 API 등 제공)
- 사용자 얼굴 임베딩 DB는 보드 내부 SPI Flash `/spiflash/faces.bin` 파일에 저장

### 2.3.2 Web UI (관리자 페이지)

- 보드 IP에 접속하여 Web UI 사용 → 예) `http://192.168.0.15`
- 화면 구성:
    - **등록 탭 (메인 화면)**
        - 영문 + 숫자 조합으로 사용자 이름 입력
        - 얼굴 검출이 되는 모습을 실시간 스트리밍 되는 화면을 보며 확인한 후 ‘등록’ 버튼을 누르면 얼굴 검출이 10 프레임 이상 유지되면 사용자 **얼굴 임베딩 + 이름 정보를 보드 내부 Flash DB(faces.bin)에 저장**
        
        ![Section 2 (27).png](attachment:9991217a-a262-4cd1-999c-76310a9b3306:cfadbc70-213f-4fca-a303-d0d6028fcd21.png)
        
    - **삭제 탭**
        - 사용자 이름 검색 후 이름으로 삭제 가능
        - 삭제 시 ESP32가 해당 사용자의 레코드를 DB에서 제거
        
        ![Section 3 (6).png](attachment:2e70de5d-30b7-461a-ba67-5535676fdefc:2319475d-7019-43ff-babf-5d3fc5e9481b.png)
        
    - **리스트 탭**
        - 현재 보드 내부 Flash DB(faces.bin)에 저장된 모든 사용자 정보를 기반으로 리스트 표시
        - 각 사용자 이름 옆에 삭제 버튼으로 사용자 삭제도 가능
        
        ![Section 4 (2).png](attachment:b401524b-3458-4ea8-a821-e5a3208b2638:c58742cd-1989-4a4d-a6e8-9e3a81c38743.png)
        
    - 출입 로그 탭
        - 출입 인증이 일어날 때마다 ESP32가 서버 API로 로그 전송 (이름+출입 시간+Threshold)
        - Threshold
            - ESP-WHO 내부 딥러닝 얼굴 검출 모델은 이 영역이 얼굴일 확률(0~1) 값을 내놓는데 threshold 는 이걸 얼굴로 볼 지 말 지 정하는 임계값
            
            ```jsx
            if (face_score > threshold) {
            얼굴이다 → 초록 박스 그림
            } else {
            얼굴 아님 → 무시
            }
            ```
            
        
        ![Section 5 (3).png](attachment:c2906f4a-017f-479b-b76e-1d009f1cbfec:0422b1f3-c424-4d87-bd7c-577b1adac694.png)
        

### 2.3.3 서버 (DB 저장용 API)

### 구성

- FastAPI 기반
    - Swagger UI `http://192.168.0.20:8000/docs`
    
    ![image.png](attachment:88634c4b-d49d-4476-9fb8-a10b9fd71e74:image.png)
    
- PostgreSQL DB

### 역할

- 출입 로그 수신/저장
    - ESP32-S3가 출입 인증 발생 시 HTTP POST로 전송한 데이터를 수신
        - `device_id` (어느 단말기/출입문인지)
        - `timestamp` (출입 시각)
        - `face_id` (보드 내부 DB의 사용자 번호)
        - `face_name` (인식된 이름)
        - `result` (성공/실패)
        - `threshold` (그 시점에 사용된 임계값
- 얼굴 DB 복구 가능하도록 임베딩 정보 저장
    - 펌웨어 업데이트 시 등록해둔 얼굴 정보가 날라갈 수 있으므로 최근 등록/삭제 된 사항들을 반영한 단말기 내부에 저장되어 있는 얼굴 임베딩 정보를 저장

## 3. 개발 진행 과정

### 3.1 v0.1~v0.3- 스트리밍 + 얼굴 검출 + 얼굴 박스 오버레이

[25.11.25 (시스템 아키텍처 정리 ,얼굴 인식 - esp-who(espressif idf 5.4버전) v0.1)](https://www.notion.so/25-11-25-esp-who-espressif-idf-5-4-v0-1-2b6d4b8233fa808a90f2ddcbb94cc7fc?pvs=21) 

[25.11.26 (esp-who v0.2)](https://www.notion.so/25-11-26-esp-who-v0-2-2b7d4b8233fa803cab65e0ea43e49158?pvs=21) 

[25.11.27 (esp-who v0.2 v0.3)](https://www.notion.so/25-11-27-esp-who-v0-2-v0-3-2b8d4b8233fa802fa487d46374d7126d?pvs=21) 

1. **실시간 스트리밍 구현 방식**
    - ESP32-S3 + OV2640 카메라 사용
    - 카메라 프레임을 MJEPG 스트리밍 방식으로 전송
        - 동영상 파일을 만들어 전송하는 것이 아닌 사진(JPEG)를 아주 빠르게 계속 보내는 방식
        - ESP32는 PC처럼 성능이 충분하지 않기 때문에 동영상 인코딩은 부담이 될 수 있음
2. **얼굴 검출 로직 구성**
    - ESP-WHO 프레임워크를 사용하여 CNN 기반 얼굴 검출 모델 사용
3. **초록 박스 표시 방식**
    - 검출된 얼굴 좌표를 ESP32 에서 계산 후 전송하면 스트리밍 서버 코드에서 박스 오버레이 처리를 하여 검출된 얼굴 위치에 초록색 사각형 표시
    - 인식 여부를 시각적으로 확인하기 위함

![스트리밍](attachment:4828061a-5812-4122-85ac-ab33f60b6737:image.png)

스트리밍

[얼굴 검출](attachment:273a365d-e9ab-476d-bac8-83290615b56c:KakaoTalk_20251126_141717509_(1).mp4)

얼굴 검출

![Section 1 (7).png](attachment:00927e51-fe65-4db6-af92-af47bfb5ae9f:68db144f-e26e-4712-8bde-e65cc8067ff9.png)

![Section 1 (8).png](attachment:e805d5ea-1b85-46ba-a3f3-47ac93994616:39cba20e-028b-44f5-b7ce-56e00db74c73.png)

### 3.2 v0.4 - 임베딩 기반 얼굴 등록/인식 + SPI Flash 저장

[25.11.28 (esp-who v0.4)](https://www.notion.so/25-11-28-esp-who-v0-4-2b9d4b8233fa80f7b127f4609de51d6f?pvs=21) 

[25.12.02 (esp-who v0.4 v0.5)](https://www.notion.so/25-12-02-esp-who-v0-4-v0-5-2bdd4b8233fa807ab6ffecf234625e46?pvs=21) 

1. **사용자 등록 방식**
    - 등록 탭에서 실시간 스트리밍 화면으로 사용자 얼굴 검출이 되고 있는지 확인 후 등록할 이름을 입력하고 얼굴 검출이 10프레임 이상 가능하도록 카메라 앞에 서서 등록 버튼 누름
    - 사용자 이름 + 얼굴 임베딩 값이 DB 에 저장
2. **사용자 인식**
    - 카메라에 얼굴이 검출되면 현재 얼굴 특징 값과 DB에 저장된 특징 값을 비교
    - 얼마나 비슷한지 점수를 계산
    - 점수가 기준보다 좋으면 같은 사람으로 판단
        - 기준값: Threshold (0~1)

---

- **얼굴 등록 전**

![image.png](attachment:025b4bab-2900-41d8-b5cc-3a6061b000e9:a8c64b0a-5006-4aa2-a96d-4a841c7378d9.png)

- **얼굴 등록**

![image.png](attachment:fde9fb45-f97a-4a5e-87f9-47364c57bfb6:15b65660-c070-44f7-b5db-5dc9efc66751.png)

- **다른 얼굴 인식 시 → “UNKNOWN” or 다른 등록된 이름으로 표시**

![image.png](attachment:8db1b77f-260a-40eb-8fec-9cb96a978b44:601db056-6234-422a-9fd3-7ddfa2baaa6e.png)

![image.png](attachment:4e1bb1ad-7386-461b-a690-b82ffaf6c0ef:996c37d7-53a8-4362-94f5-a10911170e84.png)

### 3.3 v0.5 - 사용자 목록/삭제 API + 관리 UI 확장

[25.12.02 (esp-who v0.4 v0.5)](https://www.notion.so/25-12-02-esp-who-v0-4-v0-5-2bdd4b8233fa807ab6ffecf234625e46?pvs=21) 

1. **얼굴 정보가 저장되는 구조** 
    - RAM(메모리) 안에 있는 목록
        - 보드가 켜지면, 등록된 사람들을 벡터(배열 같은 것)에 들고 있음
        - 각 사람은
            - 이름(예: `"person1"`)
            - 얼굴 임베딩(숫자 벡터)  이런 정보를 가지고 있음
    - Flash 안에 있는 파일 (`faces.bin`)
        - 전원을 꺼도 안 날아가게 SPI Flash에 `faces.bin` 파일로 한 번 더 저장해둠
        - 실행할 때 한 번 이 파일을 읽어서 → 메모리(RAM)로 가져옴
            
            (그래서 삭제/등록이 모두 이 메모리 목록을 기준으로 동작함)
            
2. **사용자 삭제 API**
    - 특정 사용자를 보드 DB에서 제거
3. **사용자 리스트 API**
    - 현재 등록된 사용자 확인

---

- **리스트 탭**
    
    ![image.png](attachment:46e86687-50fc-41ff-ac84-7b35153747ae:image.png)
    
- **리스트 탭에서 ‘삭제’ 버튼으로 ID 삭제**
    
    ![image.png](attachment:c9546df6-5443-4d63-8fe2-ccddd862c055:image.png)
    
- **삭제 후 갱신된 리스트 목록**
    
    ![image.png](attachment:664d6cba-2b1e-4d64-8a72-c6a63e5f5cae:image.png)
    

### 3.4 v0.6 - 인증 상태머신 + WS2812 LED 표시

[25.12.03 (esp-who v0.6)](https://www.notion.so/25-12-03-esp-who-v0-6-2bed4b8233fa808cb6fec358d1f3ff50?pvs=21) 

### **사용 LED**

- WS2812 네오픽셀1채널 LED 모듈 — https://item.gmarket.co.kr/Item?goodscode=2831405423&lcd=100000055&NaPm=ct%3Dmfgj0wns%7Cci%3D602f69902f43dd52f2587bda449612b1abe89f2b%7Ctr%3Dslsc%7Csn%3D24%7Chk%3D55a3207f8b899325e0069ca271984d97c047ed34&jaehuid=200001169

### LED 실험 영상

[KakaoTalk_20251226_105121799.mp4](attachment:1df1c1ea-a548-47cd-91e8-1cba34810ab9:KakaoTalk_20251226_105121799.mp4)

### 상태 정의

- **IDLE**
    - 조건: 얼굴 0명
    - 의미: 아무도 없음, 완전 대기 상태
    - LED: OFF
- **MULTI**
    - 조건: 얼굴 2명 이상
    - 의미: 다인원, 인증 대상 모호 → 인증 시도 안 함
    - LED: **IDLE과 동일하게 OFF**
    - 내부 상태만 따로 관리 (다인원으로 인한 미인증)
- **CANDIDATE**
    - 조건: **얼굴 1명**이 **연속 ≥ N프레임**(CANDIDATE_STREAK_FRAMES) 유지
        - 코드 상 10 프레임으로 설정
        - `static const int CANDIDATE_STREAK_FRAMES = 10;    // 10프레임 연속 단일 인원`
    - 의미: 단일 인원 + 안정적으로 잡힘 → 인증 대상 후보
    - LED: 주항
- **AUTHENTICATING**
    - 조건: CANDIDATE 직후, 실제 매칭 로직 실행 중
    - 의미: 지금 임베딩 매칭 중
    - 최소 유지 시간: **AUTHENTICATING_MIN_MS (≥ 500ms)**
        
        → 너무 빨리 SUCCESS/FAIL로 튀지 않게 시각적으로 보장
        
    - LED: 주항 빠른 깜빡임
        - ON 100ms / OFF 100ms (총 주기 200ms)
- **SUCCESS**
    - 조건: AUTHENTICATING 이후 **등록자 & 유사도 ≥ threshold**
        - threshold: 인식된 사람을 등록된 사람으로 인정할지 말지 결정하는 기준 (0~1)
            - 얼굴 임베딩 간 코사인 유사도를 계산한 값
        - `const float THRESH = 0.60f; // 인식 임계값`
    - 의미: 인증 성공 (문 열림, 또는 통과)
    - 유지 시간: **RESULT_HOLD_MS ≈ 3초** 고정 유지
    - LED: **초록 고정 ON (약 3초) → 이후 OFF + COOLDOWN**
- **FAIL**
    - 조건:
        - AUTHENTICATING 이후 매칭 실패
        - 또는 DB가 비어 있는데 얼굴이 있음(“등록 안 된 사람만 있음”)
    - 의미: 인증 실패 / 미등록자
    - 유지 시간: **RESULT_HOLD_MS ≈ 3초**
    - LED: **빨강 3회 깜빡(약 300ms ON / 300ms OFF) → OFF**
- **COOLDOWN**
    - 조건: SUCCESS or FAIL 이후
    - 의미: **1~2초 정도 재인증 금지 구간**
        - 카메라는 초당 10~30 프레임을 계속 보내기 때문에 cooldown 이 없으면 1번 입장했는데 여러 로그가 찍힘 또는 인증 직후 순간적으로(얼굴 좌표가 흔들리거나 고개를 돌리거나 등) FAIL 이 튀는 경우도 많이 발생
    - 시간: **COOLDOWN_MS**
    - 효과: SUCCESS/FAIL 직후 흔들리는 프레임에 재인증이 꼬이지 않게 함
    - LED: OFF

![image.png](attachment:b950dd25-95f1-4a62-86d3-104a2e3d72c5:image.png)

### 3.5 v0.7 - 서버 출입 로그 저장

[25.12.04 (esp-who v0.7)](https://www.notion.so/25-12-04-esp-who-v0-7-2bfd4b8233fa8079b8aef591adaa8e6f?pvs=21) 

[25.12.05 (esp-who v0.7)](https://www.notion.so/25-12-05-esp-who-v0-7-2c0d4b8233fa80b69617c9e23f1d7b83?pvs=21) 

[25.12.08 (esp-who v0.7 오류 해결 / 코멘트 정리)](https://www.notion.so/25-12-08-esp-who-v0-7-2c3d4b8233fa80a48704e68e24274886?pvs=21) 

### **FastAPI 서버 구축**

### 목적

- 출입 로그를 DB에 저장
- 얼굴 DB 파일 (faces.bin)을 서버로 백업

### 기술 스택

- FastAPI (Python)
- PostgreSQL (DB)

### DB 설계

1. **테이블: access_logs**
- 서버가 저장하는 원본 로그 테이블

| 컬럼명 | 타입 | 설명 |
| --- | --- | --- |
| **id** | Integer | 로그 한 건의 고유 번호 최신순 정렬 기준 |
| **device_id** | String | 어느 출입문/기기에서 발생한 로그인지 구분 |
| **face_id** | Integer | 기기 내부 얼굴 DB에서의 사용자 번호(미등록/실패 시 없음) |
| **face_name** | String | 인식된 사람 이름(미등록/실패 시 없음) |
| **event_time** | DateTime | 출입 이벤트 발생 시각(기기 또는 서버 기준 전달값) |
| **result** | String | 인증 결과(예: SUCCESS / FAIL) |
| **similarity** | Float | 얼굴 유사도 점수 |
| **threshold** | Float | 인증 통과 기준값(이 값 이상이면 성공 판단) |
1. **View: access_logs_view**
- 조회용으로 view를 사용 (Web UI 출입 로그에 표시되는 항목)
- `id, face_name, event_time_kr, threshold`

### API 설계

### 1. 출입 로그 저장 API `POST /api/access-logs`

- ESP32가 출입 인증 결과를 서버에  남김

**요청 데이터(AccessLogCreate)**

- `device_id` : 어떤 기기인지
- `event_time` : 이벤트 발생 시각
- `result` : 성공/실패 결과
- `face_id` : 얼굴DB 사용자 번호
- `face_name`  : 사람 이름
- `similarity` : 유사도 점수
- `threshold`  : 기준값

### 2. 출입 로그 조회 API `GET /api/access-logs`

- 출입로그 탭에서 최근 출입 기록을 확인

**특징**

- `access_logs_view`에서 조회
- 최근 100건을 최신순(id DESC)으로 응답

**응답 모델(AccessLogItem)**

- `id`
- `face_name`
- `event_time_kr` (한국시간 문자열)
- `threshold`

### 3. 얼굴DB 백업 업로드 API `POST /api/faces-db/{device_id}`

- ESP32가 가지고 있는 얼굴DB 파일(`faces.bin`)을 서버에 **통째로 업로드(백업)**
- `application/octet-stream` → 바이너리 파일 그대로 전송

### 4. 얼굴DB 다운로드 API `GET /api/faces-db/{device_id}`

- 서버에 저장된 `faces.bin`을 기기가 다시 내려받아 **복원**하기 위함 용

### 출입 로그 DB 연동

- 보드 → 서버 : 인증이 ‘SUCCESS’ 시 로그 전송

```jsx
AccessLog: POST http://192.168.0.20:8000/api/access-logs
body={
"device_id":"LAB",
"event_time":"1970-01-01T00:23:12+0000",
"result":"SUCCESS",
"face_id":0,
"face_name":"yh",
"similarity":0.7307
}
```

- 서버는:
    - `POST /api/access-logs` : ESP32가 보내는 출입 로그를 DB에 저장
    - `GET /api/access-logs` : 최근 로그 목록을 JSON으로 반환
    - `/docs` : Swagger UI에서 GET/POST 동작을 눈으로 확인
- 관리자용 Web UI는:
    - 출입 로그 탭에서 `GET /api/access-logs` 결과를 테이블로 표시
    
    ![image.png](attachment:9f805264-30e1-4ce2-8a3e-d774d9cf4297:image.png)
    

### 3.7 v0.8~v1.1 - 운영 편의 기능 추가

[25.12.09 (v0.8 v0.9 v1.0 v1.1)](https://www.notion.so/25-12-09-v0-8-v0-9-v1-0-v1-1-2c4d4b8233fa8023b52be4c2685882ec?pvs=21) 

1. **Threshold 슬라이더 + 서버 저장**
    - 얼굴 인식 정확도를 사용자가 Web UI 내에서 조절할 수 있도록 함
        - 이전에는 코드에 `Threshold = 0.60` 식으로 고정 시켜놔서 실험하기 불편한 점이 있었음
    - 조절한 threshold 값을 NVS(영구 저장소)에 저장하고 저장된 threshold 값으로 얼굴 인증 로직을 실행함
    - 인증 성공 시 서버로 보내는 출입 기록에 threshold 값도 포함되도록 함
    
    ![image.png](attachment:b0b95c56-15b5-4051-b3cc-ce1c29c37a22:image.png)
    
    ![image.png](attachment:048b071e-bf08-464b-a677-b51eeac6132e:image.png)
    
2. **Web UI 에 출입자 인증 상태 출력**
    - 기존에는 출입 로그 탭에서만 출입 기록을 확인할 수 있었음
    - 실제 사용을 하며 스트리밍 화면만 켜놓고 모니터링 하는 경우가 많아 실시간 화면 가까이에 ‘누가 방금 인증되었는지’ 바로 확인할 필요를 느낌
    - 스트리밍 화면 아래에 최근 출입 인증 정보를 바로 보여줄 수 있도록 함
    
    ![image.png](attachment:fe6e530a-fd32-4df5-8d05-57b83b9fa20f:image.png)
    
3. **얼굴 정보 (faces.bin) 서버에 저장하여 백업**
    - 펌웨어가 업데이트 되면 현재까지 보드 flash에 저장되어 있던 얼굴 임베딩 정보가 날라가는 한계점이 있었음
    - 얼굴 등록/삭제가 일어났을 때마다 DB 전체를 다시 faces.bin 파일로 저장하고 서버로 전송함
    - ‘서버에서 얼굴 DB 가져오기’ 버튼 클릭 시:
        - 웹 UI → `/faces_restore` 로 요청 전송
        - 보드가 서버로 요청: `GET /api/faces-db/LAB`
        - 서버가 저장해둔 faces.bin 다운로드
        - 보드는 파일을 `/spiflash/faces.bin`으로 교체
        - DB를 다시 메모리에 로드
        - 보드의 얼굴 정보가 서버 백업을 기준으로 완전히 복구됨 → 즉시 인식에 반영됨
    - 서버에 `/spiflash/faces.bin`이 없으면 → `복구 실패(500): download failed`
    
    ![image.png](attachment:717a6a2a-2357-43ae-bb7d-209cbf560feb:image.png)
    

### 3.8 서보모터 조립 및 PWM 제어 테스트

[25.12.10 (서보모터 조립 및 테스트)](https://www.notion.so/25-12-10-2c5d4b8233fa80c8b94ecb3b1b5a334c?pvs=21) 

- 사용 서보모터: https://www.coupang.com/vp/products/7206616492?itemId=18226764638&vendorItemId=85374302199&q=%EC%84%9C%EB%B3%B4%EB%AA%A8%ED%84%B0+%ED%8C%AC%ED%8B%B8%ED%8A%B8&searchId=138383cc2420196&sourceType=search&itemsCount=36&searchRank=1&rank=1&traceId=mislalu0
- 조립 참고 영상: https://www.youtube.com/watch?v=cgFVEk7z46U
- 조립 완성 모습:

![image.png](attachment:6a852b4d-d56d-4b77-a1c6-04da235e948e:image.png)

![image.png](attachment:b1f5b578-84a1-4f16-8707-6cb482f2aa37:image.png)

![image.png](attachment:0400ac40-6360-44ad-820b-d91b2194bfd1:image.png)

![image.png](attachment:d87e99fe-279f-4e32-a044-1bfba69bad7c:image.png)

### 3.8.1 SG90 서보모터 기본 정보

### 스펙

- 무게: 17g
- 속도: 100ms/60도 (4.8V 기준)
- 동작 각도: 90도
- 동작 전압: 4.8V ~ 6V

### 케이블

- 주항색: Signal
- 빨간색: VCC (5V)
- 갈색: GND

### 3.8.2 팬(Pan) / 틸트(Tilt) 모터 구조

![Section 2 (13).png](attachment:ae835d1d-e519-4f43-870d-268b97daceed:7ec489e8-1c5d-4f77-bf4c-764610e75b5d.png)

### 팬(Pan)

- 아래쪽 서보 (베이스에 고정됨)
- 카메라를 좌 ↔ 우 회전시키는 역할

### 틸트(Tilt)

- 위쪽 서보 (팬 서보 위에 결합 됨)
- 카메라를 위 ↔ 아래로 움직이는 역할

### 3.8.3 전원 구성

- 서보는 전류를 많이 먹기 때문에 전원 구성에 주의해야 함
- 테스트 단계에서는
    - USB 5V 만으로 ESP32 + 서보 2개 연결 가능
    - 단 서보가 강하게 버벅일 수 있음
- 실제 사용 단계에서는
    - 외부 5V 1A~2A 전원 사용 권장
    - 공통 GND 필수

### 3.8.4 ESP32 와 연결 방식

| 서보 | 신호핀 | 전원(GND/5V) |
| --- | --- | --- |
| 팬(Pan) | ESP32 GPIO 13 (예) | 5V, GND |
| 틸트(Tilt) | ESP32 GPIO 14 (예) | 5V, GND |

- GPIO 번호는 변경 가능 
(PWM 출력 가능한 핀 내에서)

### 3.8.5 서보모터 Test

### 코드의 목적

- 서보 2개(팬, 틸트)를 같은 타이머(50Hz PWM) 에 물려서 제어
- 초기엔 둘 다 중앙 위치 → 틸트(위아래) 왔다갔다 2번 → 팬(좌우) 왔다갔다 2번 반복
- 코드
    
    ```cpp
    #include "driver/ledc.h"
    #include "esp_log.h"
    #include "freertos/FreeRTOS.h"
    #include "freertos/task.h"
    
    static const char *TAG = "SERVO_TEST";
    
    #define SERVO_PAN_GPIO   GPIO_NUM_13   // 아래쪽 팬 서보
    #define SERVO_TILT_GPIO  GPIO_NUM_14   // 위쪽 틸트 서보
    
    #define SERVO_FREQ       50
    #define SERVO_TIMER      LEDC_TIMER_0
    #define SERVO_MODE       LEDC_LOW_SPEED_MODE
    #define SERVO_PAN_CH     LEDC_CHANNEL_0
    #define SERVO_TILT_CH    LEDC_CHANNEL_1
    
    // 각도 → 듀티 변환
    static uint32_t angle_to_duty(float deg)
    {
        if (deg < 0.0f)   deg = 0.0f;
        if (deg > 180.0f) deg = 180.0f;
    
        const float SERVO_MIN_US = 600.0f;
        const float SERVO_MAX_US = 2400.0f;
        const float PERIOD_US    = 20000.0f;  // 20ms @ 50Hz
    
        float us = SERVO_MIN_US +
                   (SERVO_MAX_US - SERVO_MIN_US) * (deg / 180.0f);
    
        float max_duty = (1 << LEDC_TIMER_14_BIT) - 1;
        return (uint32_t)((us / PERIOD_US) * max_duty + 0.5f);
    }
    
    static void set_servo(float pan_deg, float tilt_deg)
    {
        uint32_t duty_pan  = angle_to_duty(pan_deg);
        uint32_t duty_tilt = angle_to_duty(tilt_deg);
    
        ledc_set_duty(SERVO_MODE, SERVO_PAN_CH, duty_pan);
        ledc_update_duty(SERVO_MODE, SERVO_PAN_CH);
    
        ledc_set_duty(SERVO_MODE, SERVO_TILT_CH, duty_tilt);
        ledc_update_duty(SERVO_MODE, SERVO_TILT_CH);
    
        ESP_LOGI(TAG, "pan=%.1f tilt=%.1f", pan_deg, tilt_deg);
    }
    
    extern "C" void app_main(void)
    {
        ESP_LOGI(TAG, "Servo test start");
    
        // 1) 타이머 설정
        ledc_timer_config_t timer_cfg = {};
        timer_cfg.speed_mode      = SERVO_MODE;
        timer_cfg.timer_num       = SERVO_TIMER;
        timer_cfg.duty_resolution = LEDC_TIMER_14_BIT;
        timer_cfg.freq_hz         = SERVO_FREQ;
        timer_cfg.clk_cfg         = LEDC_AUTO_CLK;
        ESP_ERROR_CHECK(ledc_timer_config(&timer_cfg));
    
        // 2) PAN 채널 (GPIO 13)
        ledc_channel_config_t pan_cfg = {};
        pan_cfg.gpio_num   = SERVO_PAN_GPIO;
        pan_cfg.speed_mode = SERVO_MODE;
        pan_cfg.channel    = SERVO_PAN_CH;
        pan_cfg.intr_type  = LEDC_INTR_DISABLE;
        pan_cfg.timer_sel  = SERVO_TIMER;
        pan_cfg.duty       = 0;
        pan_cfg.hpoint     = 0;
        ESP_ERROR_CHECK(ledc_channel_config(&pan_cfg));
    
        // 3) TILT 채널 (GPIO 14)
        ledc_channel_config_t tilt_cfg = {};
        tilt_cfg.gpio_num   = SERVO_TILT_GPIO;
        tilt_cfg.speed_mode = SERVO_MODE;
        tilt_cfg.channel    = SERVO_TILT_CH;
        tilt_cfg.intr_type  = LEDC_INTR_DISABLE;
        tilt_cfg.timer_sel  = SERVO_TIMER;
        tilt_cfg.duty       = 0;
        tilt_cfg.hpoint     = 0;
        ESP_ERROR_CHECK(ledc_channel_config(&tilt_cfg));
    
        // 각도 프리셋
        const float CENTER     = 30.0f;  // 중앙
        const float TILT_UP    = 10.0f;  // 위쪽
        const float TILT_DOWN  = 100.0f; // 아래쪽
        const float PAN_LEFT   = 30.0f;  // 왼쪽
        const float PAN_RIGHT  = 120.0f; // 오른쪽
    
        while (true) {
            // 0) 둘 다 중앙
            set_servo(CENTER, CENTER);
            vTaskDelay(pdMS_TO_TICKS(1500));
    
            // 1) 위/아래 (틸트만) 2번
            for (int i = 0; i < 2; ++i) {
                set_servo(CENTER, TILT_UP);
                vTaskDelay(pdMS_TO_TICKS(1000));
    
                set_servo(CENTER, TILT_DOWN);
                vTaskDelay(pdMS_TO_TICKS(1000));
            }
    
            // 다시 중앙
            set_servo(CENTER, CENTER);
            vTaskDelay(pdMS_TO_TICKS(1000));
    
            // 2) 좌/우 (팬만) 2번
            for (int i = 0; i < 2; ++i) {
                set_servo(PAN_LEFT, CENTER);
                vTaskDelay(pdMS_TO_TICKS(1000));
    
                set_servo(PAN_RIGHT, CENTER);
                vTaskDelay(pdMS_TO_TICKS(1000));
            }
    
            // 다시 중앙
            set_servo(CENTER, CENTER);
            vTaskDelay(pdMS_TO_TICKS(1500));
        }
    }
    ```
    
    ### 1. 핀/타이머/채널 정의
    
    ```jsx
    #define SERVO_PAN_GPIO   GPIO_NUM_13   // 아래쪽 팬 서보
    #define SERVO_TILT_GPIO  GPIO_NUM_14   // 위쪽 틸트 서보
    ```
    
    - PAN 서보 = GPIO13, TILT 서보 = GPIO14
    
    ```jsx
    #define SERVO_FREQ       50   // 50Hz PWM
    #define SERVO_TIMER      LEDC_TIMER_0  // 타이머
    #define SERVO_MODE       LEDC_LOW_SPEED_MODE
    #define SERVO_PAN_CH     LEDC_CHANNEL_0
    #define SERVO_TILT_CH    LEDC_CHANNEL_1
    ```
    
    - PMW(Pulse Width Modulation) = 펄스 폭 변조
        - HIGH(전원 ON) 시간을 얼마나 길게, 얼마나 짧게 줄 것인지 조절하는 방식
        - 50Hz → 1초에 50번 (20ms) 마다 새로운 각도 명령 (PWM 펄스)을 보내는 방식
    - LEDC = LED Controller
        - 타이머(LEDC_TIMER_x)
            - 해당 PWM 그룹의 공통 설정 담당 ex) 주파수, 분해능 등
            - LEDC_TIMER_0~3 까지 있음
                - 각 다른 종류의 PWM을 만들 때 사용
                - 예) 타이머 0: 서보모터, 타이머 1: LED 밝기 조절 등
        - 채널(LEDC_CHANNEL_x)
            - 각 GPIO 하나하나를 제어하는 역할
            - 어느 핀에서 PWM 나갈지, 그 핀의 duty(각도) 값
            - 채널 0 → 팬 서보 용 / 채널 1 → 틸트 서보 용 으로 사용
        - 여러 채널이 하나의 타이머를 공유할 수 있음 
        → 주파수, 분해능은 같고 / duty는 채널 별로 다르게
    - SERVO_MODE
        - LEDC_HIGH_SPEED_MODE
            - 더 빠른 응답 / 내부 고속 클럭 기반
        - LEDC_LOW_SPEED_MODE
            - 느린 변경도 허용 / 일부 핀 제한
    
    ### 2. angle_to_duty() — 각도를 PWM 펄스로 바꾸기
    
    ```jsx
    static uint32_t angle_to_duty(float deg)
    {
    		if (deg < 0.0f)   deg = 0.0f;
    		if (deg > 180.0f) deg = 180.0f;
    
        const float SERVO_MIN_US = 600.0f;
        const float SERVO_MAX_US = 2400.0f;
        const float PERIOD_US    = 20000.0f;  // 20ms @ 50Hz
        
        float us = SERVO_MIN_US +
               (SERVO_MAX_US - SERVO_MIN_US) * (deg / 180.0f);
        float max_duty = (1 << LEDC_TIMER_14_BIT) - 1;
        return (uint32_t)((us / PERIOD_US) * max_duty + 0.5f);
    }
    ```
    
    1. 클램프
        - deg 가 0~180도 범위를 넘지 못하게 막는 것
    2. 펄스 범위 정의
        - 서보는 **50Hz = 주기 20ms**의 PWM 신호를 받음
        - 이 20ms 중에서:
            - 펄스가 HIGH로 유지되는 시간(µs 단위)에 따라 각도가 결정됨
        - 일반적인 서보:
            - 약 0.5ms(500µs) 근처 → 최소 각도
            - 약 1.5ms(1500µs) → 중앙
            - 약 2.5ms(2500µs) 근처 → 최대 각도
        - 이 코드는 그걸
            - **600µs ~ 2400µs** 로 쓰고 있는 것
            - 즉, 우리 기준:
                - 0도 → 600µs
                - 180도 → 2400µs
        1. 각도 → 펄스 변환
            - deg를 0~180 사이 비율로 바꾼 다음 그 비율만큼 600~2400µs 구간에서 보간
        2. 펄스 → duty 값 변환
            - LEDC는 내부적으로 **0 ~ (2^N - 1)** 카운터로 PWM을 만듦
                - 여기서는 `N = 14bit` → `0 ~ 16383`
            - `us / PERIOD_US` = **펄스 비율** (20ms 중 몇 %인가)
            - 그 비율에 `max_duty`를 곱해서 하드웨어가 이해하는 duty 값으로 변경
    
    ### 3. set_servo() — 실제 두 서보에 적용
    
    ```jsx
    static void set_servo(float pan_deg, float tilt_deg)
    {
    uint32_t duty_pan  = angle_to_duty(pan_deg);
    uint32_t duty_tilt = angle_to_duty(tilt_deg);
    ledc_set_duty(SERVO_MODE, SERVO_PAN_CH, duty_pan);
    ledc_update_duty(SERVO_MODE, SERVO_PAN_CH);
    
    ledc_set_duty(SERVO_MODE, SERVO_TILT_CH, duty_tilt);
    ledc_update_duty(SERVO_MODE, SERVO_TILT_CH);
    
    ESP_LOGI(TAG, "pan=%.1f tilt=%.1f", pan_deg, tilt_deg);
    }
    ```
    
    - PAN, TILT 각도 각각을 angle_to_duty 로 변환
    - 그 duty를 서보 채널에 반영
    
    ### 4. app_main() — 초기 설정 + 동작 시나리오
    
    **(1) 타이머 설정**
    
    ```
    ledc_timer_config_t timer_cfg = {};
    timer_cfg.speed_mode      = SERVO_MODE;
    timer_cfg.timer_num       = SERVO_TIMER;
    timer_cfg.duty_resolution = LEDC_TIMER_14_BIT;
    timer_cfg.freq_hz         = SERVO_FREQ;
    timer_cfg.clk_cfg         = LEDC_AUTO_CLK;
    ESP_ERROR_CHECK(ledc_timer_config(&timer_cfg));
    ```
    
    - 타이머의 역할은 이 PWM 그룹의 주파수(=서보용 50Hz)와 분해능(=몇 비트로 duty 쪼갤지) 정함
    - 이 타이머에 붙은 채널들은 모두 이 설정을 공유
    
    **(2) 채널 설정 (PAN/TILT)**
    
    ```
    ledc_channel_config_t pan_cfg = {};
    pan_cfg.gpio_num   = SERVO_PAN_GPIO;
    pan_cfg.speed_mode = SERVO_MODE;
    pan_cfg.channel    = SERVO_PAN_CH;
    pan_cfg.intr_type  = LEDC_INTR_DISABLE;
    pan_cfg.timer_sel  = SERVO_TIMER;
    pan_cfg.duty       = 0;
    pan_cfg.hpoint     = 0;
    ESP_ERROR_CHECK(ledc_channel_config(&pan_cfg));
    
    ledc_channel_config_t tilt_cfg = {};
    tilt_cfg.gpio_num   = SERVO_TILT_GPIO;
    tilt_cfg.speed_mode = SERVO_MODE;
    tilt_cfg.channel    = SERVO_TILT_CH;
    tilt_cfg.intr_type  = LEDC_INTR_DISABLE;
    tilt_cfg.timer_sel  = SERVO_TIMER;
    tilt_cfg.duty       = 0;
    tilt_cfg.hpoint     = 0;
    ESP_ERROR_CHECK(ledc_channel_config(&tilt_cfg));
    ```
    
    - PAN: GPIO 13에 PWM 뿌리는 채널 하나 생성
    - TILT: GPIO 14에 PWM 뿌리는 채널 하나 생성
    - 둘 다 **만든 타이머(50Hz, 14bit)** 를 공유
    
    **(3) 각도 프리셋**
    
    ```
    const float CENTER     = 30.0f;  // 중앙
    const float TILT_UP    = 10.0f;  // 위쪽
    const float TILT_DOWN  = 100.0f; // 아래쪽
    const float PAN_LEFT   = 30.0f;  // 왼쪽
    const float PAN_RIGHT  = 120.0f; // 오른쪽
    ```
    
    - 실제 물리적 중앙/좌/우/위/아래에 맞춰서 임의로 잡은 값
    
    **(4) 동작** 
    
    ```
    while (true) {
        // 0) 둘 다 중앙
        set_servo(CENTER, CENTER);
        vTaskDelay(pdMS_TO_TICKS(1500));
        
        // 1) 위/아래 (틸트만) 2번
        for (int i = 0; i < 2; ++i) {
            set_servo(CENTER, TILT_UP);
            vTaskDelay(pdMS_TO_TICKS(1000));
    
            set_servo(CENTER, TILT_DOWN);
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
    
        // 다시 중앙
        set_servo(CENTER, CENTER);
        vTaskDelay(pdMS_TO_TICKS(1000));
        
        // 2) 좌/우 (팬만) 2번
        for (int i = 0; i < 2; ++i) {
            set_servo(PAN_LEFT, CENTER);
            vTaskDelay(pdMS_TO_TICKS(1000));
    
            set_servo(PAN_RIGHT, CENTER);
            vTaskDelay(pdMS_TO_TICKS(1000));
        }
    
        // 다시 중앙
        set_servo(CENTER, CENTER);
        vTaskDelay(pdMS_TO_TICKS(1500));
    }
    ```
    
    - 팬/틸트 중앙 자세 → 팬 고정, 틸트 up/down 2번 반복 → 틸트 고정, 팬 left/right 2번 반복

[KakaoTalk_20251210_175938504.mp4](attachment:eedf365b-5512-40f7-be64-d4915a28a6b8:KakaoTalk_20251210_175938504.mp4)

### 3.9 v1.2 - ESP-WHO 서보 트래킹 적용 + 설치 (프로토타입 완성)

[25.12.11 (서보모터 포인터 구현)](https://www.notion.so/25-12-11-2c6d4b8233fa80848e29d2110994a9cc?pvs=21) 

[25.12.12 (v1.3 출입인증 시스템 + 서보모터 적용)](https://www.notion.so/25-12-12-v1-3-2c7d4b8233fa80aeaa7adc2bf88f1f0d?pvs=21) 

### 3.9.1 ESP-WHO와 OpenCV의 트래킹 성능 차이

| 항목 | OpenCV (PC) | ESP-WHO (ESP32) |
| --- | --- | --- |
| 검출 방식 | Detection + Tracking | Detection만 있음 |
| 좌표 안정성 | 부드럽게 이어짐 | 매 프레임 튀는 경우 많음 |
| 얼굴 사라짐 처리 | Tracker가 예측 | 좌표가 아예 사라짐 |
| 모터 제어 적합성 | 매우 적합 | **제한된 안정성** |

### 일반적인 PC + OpenCV + 서보모터 얼굴 트래킹

1. **얼굴 검출 (Detection)**
- 얼굴이 화면 어디에 있는지 처음 찾는 역할
    - OpenCV DNN or Haar Cascade 등 활용
    - 여기서 나온 faces(가장 큰 얼굴 검출 박스)를 트래커 초기화용 ROI로 넘김
- 검출된 얼굴 사각형의 중심 좌표를 트래킹 포인트로 사용
1. **추적 (Tracking)**
- 검출은 무겁고 프레임마다 돌리면 느리거나 튀기 때문에 검출로 한 번 잡고 그 다음엔 tracker로 계속 끌고가는 패턴 사용
    - OpenCV에서 흔히 쓰는 tracker:
        - KCF: 빠르고 꽤 안정적, 가장 많이 사용
        - CSRT: 더 정확하지만 좀 느림
        - MOSSE: 매우 빠른 대신 정확도나 내구성이 좀 떨어짐
    - 새로 얼굴이 검출되지 않아도 tracker는 ‘이전 프레임에서 여기 있었으니 이번 프레임에서는 이 방향으로 조금 움직였을 것’ 라고 예측하는 식으로 박스를 계속 갱신함
- 즉, 전용 trakcer가 있어서 검출 없이도 프레임 사이 위치를 예측하고 연속성 있게 모터가 돌아감

### ESP-WHO 의 face detection 통해 제어

- OpenCV처럼 ‘검출 후 추적’ 구조가 아니라 매 프레임마다 새로운 얼굴 검출을 반복하는 방식
1. **얼굴 검출 (Detection)**
- ESP-WHO는 매 프레임 실시간 얼굴 검출 수행
- OpenCV처럼 tracker를 사용하지 않기 때문에 얼굴이 사라지거나 흔들리면 위치 예측이 불가능하고 서보의 위치가 꼬이는 등의 문제가 다수 발생할 수 있음
- 서보는 느리고 검출은 빨라 속도 매칭이 안됨
    - 서보모터는  60도 움직이는데 약 0.1초 ~ 0.2초 → 기계적 움직임 속도 제한 있음
    - ESP-WHO는 15~25fps 로 얼굴 박스를 갱신 → 0.04초 ~ 0.06초마다 새로운 좌표가 들어옴
    - 서보는 느리게 움직이는데 얼굴 좌표는 너무 빨리 바뀌기 때문에 따라가다가 계속 엇나감
        - OpenCV에서는 tracker 때문에 좌표 변화가 아주 부드럽고 예측된 값
        - ESP-WHO 에서는 매 프레임 새 검출이라 뒤죽박죽
1. **히스테리시스 (Hysteresis)**
- 최근 검출된 얼굴 박스를 N 프레임 저장해두고 1~3 프레임 정도 미검출 상태가 발생해도 이전 박스를 사용해 지속적으로 제어 (짧은 시간용 tracking-like 동작)
- ESP-WHO 는 tracker가 없어서 박스가 순간적으로 사라지면 서보가 멈추거나 튀는 문제가 발생하기 때문에 이를 보완하기 위해 히스테리시스 로직을 구현함
- 하지만, 히스테리시스는 ‘박스 튐 보정’ 용 이지 트래킹이 아니기 때문에 연속적인 트래킹 동작을 구현할 수 없음
- 또한 ESP-WHO 얼굴 박스 자체가 OpenCV 보다 훨씬 불안정하여 박스 중심좌표가 항상 흔들림

### 3.9.2 서보모터 위에 카메라를 올린 ESP-WHO 얼굴 트래킹

### 핵심 아이디어

1. **얼굴 검출 → 얼굴 중심 좌표(cx, cy) 추출**
    - ESP-WHO 에서 프레임마다 사람 얼굴을 검출하고 그 얼굴의 중심점 좌표(cx, cy) 를 받음
2. **얼굴 중심 vs 화면 중심 비교**
    - 화면 중심: (center_x, center_y)
    - 얼굴 위치: (cx, cy)
    - 오차:
        
        ```jsx
        err_x = cx - center_x
        err_y = cy - center_y
        ```
        
3. **오차를 정규화해 서보 회전값으로 변환**
    
    ```jsx
    nx = err_x / center_x   (좌우 비율)
    ny = err_y / center_y   (상하 비율)
    ```
    
4. **데드존 적용**
    - 얼굴이 거의 중앙일 때는 서보가 불필요하게 흔들리는 걸 방지
    
    ```jsx
    static const float SERVO_PAN_DEADZONE  = 0.02f;
    static const float SERVO_TILT_DEADZONE = 0.02f;
    
    if abs(nx) < DEADZONE → 0으로 처리
    if abs(ny) < DEADZONE → 0으로 처리
    ```
    
5. **서보 목표 각도 생성**
    - 각도는 서보의 최소, 최대 범위 (SERVO_MIN/MAX) 안에 제한
    
    ```jsx
    // 각도 제한 (기구적으로 안전한 범위)
    static const float SERVO_PAN_MIN  = 10.0f;
    static const float SERVO_PAN_MAX  = 120.0f;
    static const float SERVO_TILT_MIN = 10.0f;
    static const float SERVO_TILT_MAX = 120.0f;
    
    target_pan  = CENTER_PAN  + nx * PAN_RANGE
    target_tilt = CENTER_TILT + ny * TILT_RANGE
    ```
    

### Tracking 한계 보안 방식

1. **히스토리 기반 필터링**
    - 최근 8 프레임의 얼굴 중심 좌표를 저장
    - 8 프레임동안 흔들리는 값들을 모아 평균 값으로 계산해 부드럽게 추적
2. **서보 업데이트 간격 제한**
    - 프레임은 30fps 이면 초당 30번이 들어오는데 서보 모터를 1초에 30번 위치 이동 명령을 보낼 수 없기 때문에 서보 명령 빈도를 낮춰 흔들림을 줄이는 효과를 냄
        
        ```jsx
        static const int64_t SERVO_UPDATE_INTERVAL_MS = 100;  // 1초에 10번
        
        if now - last_update >= UPDATE_INTERVAL:
        ```
        
3. **이동량 제한**
    - 갑자기 튀는 얼굴 검출이나 얼굴 이동으로 인해 서보 모터가 확 움직이면 부드럽지 못하고 기계도 위험하기 때문에 서보가 갑자기 너무 크게 움직이지 않게 막음
        
        ```jsx
        // 한 번에 움직이는 최대 각도(부드럽게 따라가기)
        static const float SERVO_MAX_STEP_DEG = 3.0f;
        delta_pan = clamp(target - current, -MAX_STEP, +MAX_STEP)
        ```
        
        - 예) `80° → 77° → 74° → 71° → 68° → ... → 20°`
    - 천천히 움직이게 함으로써
        - 자연스러운 카메라 추적
        - 서보 부하 감소
        - 진동/소음 감소

### 최종 트래킹 루프 요약

```jsx
if 얼굴 없음:
	서보 업데이트 스킵
else:
	① 최근 얼굴 좌표 히스토리 갱신
	② 히스토리로 평균 위치 계산
	③ 화면 중심과의 오차 계산
	④ 데드존 적용
	⑤ 목표 pan/tilt 각도 계산
	⑥ 업데이트 간격 체크
	⑦ 한 스텝씩 부드럽게 이동 (max step 제한)
	⑧ 현재 서보 각도 저장
```

### 동작 동영상

[KakaoTalk_20251212_145034385.mp4](attachment:3f3fbed3-45da-4bd9-b2f6-cc05f07fa200:KakaoTalk_20251212_145034385.mp4)

### 3.9.3 프로토타입 완성

### 연구소 앞 설치

- 권장: 카메라 ↔ 사람 얼굴 거리 40cm ~ 50cm
- 데모 영상: https://drive.google.com/drive/folders/1FFd2yBBWBz2TWsMGH4dfdfB4-6MjLH9F?hl=ko

![image.png](attachment:b0d1bdd9-050f-4b28-85ca-56bcd3e1dc53:image.png)

![image.png](attachment:92fb113e-0974-4f21-a043-e8055329373b:image.png)

### 3.10 구현 과정에서의 문제 및 해결 과정

1. **스트리밍 실패**
    - 원인: JPEG로 보낸다고 했는데 실제 프레임은 RGB565
    - 해결: JPEG 캡처/전송 경로 확정 + 필요 시 RGB 변환 후 검출 수행
2. **WDT 리부팅(스트리밍+검출 동시 동작 불안정)**
    - 원인: 카메라 프레임을 여러 곳에서 동시에 `cam_fb_get()`하여 충돌
    - 해결: **단일 캡처 구조**로 통일

**3. DB 영구 저장 문제**

- 원인: RAM DB는 전원 OFF 시 초기화
- 해결: SPI Flash FATFS(`/spiflash/faces.bin`) 저장/로드
1. **PTZ 트래킹의 근본 한계**
    - ESP-WHO는 Detection만 있어 좌표가 튈 수 있음
    - 해결(완화):
        - 히스토리 평균
        - 데드존
        - 업데이트 간격 제한
        - 이동량 제한
    - 결론: OpenCV급 고급 트래킹은 어렵지만 **사람 방향을 대략적으로 바라보는 PTZ 수준 달성**

## 4. 평가 및 결론

### 4.1 목표 대비 성과

본 프로젝트는 **ESP32-S3 단독 환경에서 얼굴 인식 기반 출입 인증 시스템을 구현하는 것**을 목표로 하였으며, PoC(개념 검증) 단계에서 요구한 핵심 기능을 대부분 달성하였다.

- **ESP32-S3 보드 단독 얼굴 인식 시스템 구현**
    - ESP32-S3 단일 보드에서 얼굴 검출 → 얼굴 인식 → 출입 판단까지의 전 과정을 수행하도록 구현하였다.
    - 서버는 출입 로그 및 얼굴 DB 백업을 담당하며, **실시간 인식 판단에는 관여하지 않는 구조**로 설계하였다.
    - 이를 통해 네트워크 장애 상황에서도 **기본적인 출입 인증이 가능한 독립형 시스템**을 구현하였다.
- **웹 기반 사용자 관리 기능 구현**
    - Web UI를 통해 얼굴 등록, 삭제, 인식 threshold 조절 등의 관리 기능을 제공하였다.
    - 별도의 개발 지식 없이도 관리자 관점에서 시스템을 운영할 수 있도록 **단순하고 직관적인 관리 인터페이스**를 구현하였다.
- **출입 인증 기록의 체계적인 관리**
    - 출입 성공 시 얼굴 유사도(similarity)와 threshold 값을 함께 서버로 전송하여 저장하였다.
    - 이를 통해 단순한 출입 기록을 넘어, **인식 기준과 결과를 함께 분석할 수 있는 로그 구조**를 마련하였다.
- **서보모터 기반 카메라 방향 제어 실험**
    - 얼굴 위치에 따라 Pan/Tilt 서보모터를 제어하여 카메라가 사용자를 바라보도록 하는 자동 PTZ 구조를 구현하였다.
    - 고정형 카메라 대비 사용자의 위치 변화에 어느 정도 대응할 수 있는 구조를 실험적으로 검증하였다.

### 4.2 한계점 및 개선 방향

프로토타입 단계에서 의미 있는 기능을 구현하였으나, 실제 상용 환경을 고려할 경우 다음과 같은 한계점이 확인되었다.

- **환경 변화에 대한 인식 민감도**
    - 조명 방향, 밝기, 그림자 등에 따라 얼굴 인식률이 크게 변동하였다.
    - 안경 착용 여부나 마스크 착용 시 동일 인물을 다른 사람으로 인식하는 경우가 발생하였다.
    - 향후에는 조명 조건에 강인한 임베딩 모델 적용 또는 데이터 증강 기반 보완이 필요하다.
- **사용자 위치 및 각도 변화에 따른 인식 불안정**
    - 카메라 정면이 아닌 경우 인식률이 급격히 저하되는 문제가 확인되었다.
    - 얼굴 크기, 거리, 각도가 일정하지 않으면 인식 실패 빈도가 증가하였다.
    - 이를 보완하기 위해:
        - 얼굴 등록 시 다양한 각도·거리의 샘플 확보
        - 얼굴 위치를 정면으로 유도하는 UI/가이드 추가
            
            가 필요하다고 판단된다.
            
- **카메라 및 서보모터 제어의 물리적 한계**
    - 서보모터 기반 제어만으로는 얼굴을 항상 화면 중앙에 유지하는 데 한계가 있었다.
    - 현재 구조에서는 사용자가 어느 정도 카메라 위치를 맞춰야 하는 형태이다.
    - 향후에는:
        - 중앙 정면 유지 모드 도입
        - 더 정밀한 PTZ 기구 또는 추적 알고리즘 적용
            
            등을 통해 안정성을 개선할 수 있다.
            
- **얼굴 DB 관리 구조의 복잡성**
    - 현재는 `faces.bin` 파일을 서버에 백업/복원하는 방식으로 얼굴 데이터를 관리한다.
    - 이상적인 구조로는:
        - 서버에서 얼굴 DB를 직접 관리하는 구조
        - 또는 펌웨어 업데이트 시 얼굴 데이터 영역만 유지하는 방식
            
            이 필요하다.
            
- **카메라 모듈 확장성 부족**
    - 현재 구현은 OV2640 카메라 모듈에 한정되어 있다.
    - 향후 OV5640 등 다양한 해상도의 카메라를 적용하여 인식률과 화질 개선 가능성을 검토할 수 있다.

### 4.3 종합 결론

본 프로젝트는 제한된 하드웨어 자원 환경(ESP32-S3)에서 **얼굴 인식 기반 출입 인증 시스템을 실제 동작 가능한 수준으로 구현** 하였다.

PoC 단계에서는 요구사항을 대부분 충족하였지만, 실제 상용 운영을 위해서는 환경 대응력과 인식 안정성 개선이 필요하다.