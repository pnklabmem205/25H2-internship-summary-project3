실험

---

## 1. 개요

### 👥 프로젝트 정보

- 진행자: 
- 기간: 2025년 12월 15일 ~ 2026년 01월 28일
- 프로젝트 목표: **MediaPipe 기반 맨몸운동 자동 카운터 시스템 구현**
    - 카메라 영상만으로 사람의 반복 동작을 분석해 운동 횟수를 자동 측정
    - 푸쉬업(상체), 스쿼트(하체),크런치(복부),버피테스트(전신) 총 4가지 운동의 반복 횟수를 자동 카운트
    - 윈도우 데스크탑과 모바일 환경에서 구동 가능한 형태로 제작
- 깃허브: https://github.com/pnkmem432/workout-counter-pc —PC  https://github.com/pnkmem432/workout-counter-mobile —모바일
- 구글 드라이브: https://drive.google.com/drive/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo?hl=ko

---

### 💡 프로젝트 요약

본 프로젝트는 **카메라 영상으로부터 MediaPipe가 추출하는 33개 관절 랜드마크를 이용하여, 맨몸운동(스쿼트/푸쉬업/크런치/버피테스트)의 반복 동작을 자동으로 인식하고 횟수를 카운트 하는 시스템을 개발하는 것을 목표**로 하였다.

초기에는 PC 환경에서 OpenCV + MediaPipe로 실시간 포즈 추론 및 기본 카운팅 로직을 설계하고, 가상 영상(Mixamo/Blender)과 실제 촬영 영상으로 알고리즘의 동작 가능성을 검증했다. 이후 사용자가 실제로 사용하기 쉬운 형태를 목표로 모바일 앱(Flutter)로 이식하며 Android(Kotlin)에서 MediaPipe PoseLandmarker를 실행하고, Flutter 화면에 스켈레톤을 오버레이 하여 실시간 카운팅을 구현하였다.

마지막으로 로그인/회원가입, 저장/기록 조회, 설정(기록 초기화/회원 삭제), 등 부가 기능을 추가하여 개인별 운동 기록 앱 형태로 완성하고, 촬영 위치 및 조명 실험을 통해 권장 촬영 가이드를 도출한 뒤 Release APK로 결과물을 도출하였다.

---

### 🎯 목표 및 기대효과

**구체적 목표**

- **MediaPipe Pose 기반 실시간 관절 추론 파이프라인 구축**
    - MediaPipe Pose를 활용하여 영상 프레임마다 33개 관절 랜드마크를 안정적으로 추출할 수 있는 처리 흐름을 구성
    - PC와 모바일(Android) 환경 모두에서 실시간 추론이 가능하도록 연동 구조를 구현
- **맨몸운동 4종에 대해 명확히 정의된 카운팅 로직 설계 및 구현**
    - 스쿼트, 푸쉬업, 크런치, 버피테스트 각각에 대해 수치 기반 + 상태 머신(FSM) 형태로 카운팅 로직을 명확히 정의
    - 각 운동이 실제 사람 동작에서 연속적으로 안정적으로 카운트되도록 임계값 등을 적용
    - 사람의 키, 카메라 거리 차이에 따라 절대 좌표가 달라지는 문제를 해결하기 위해 몸통 길이, 어깨 폭 등 신체 기준 길이를 이용한 정규화 로직을 적용
- **촬영 환경(카메라 위치·조명)에 따른 성능 차이 실험 및 가이드 도출**
    - 카메라 높이(바닥/65cm/90cm)와 조명(light/dark)을 변경한 조건에서 카운트 정확도와 포즈 품질을 비교 실험하여 운동 카운팅에 가장 안정적인 촬영 환경을 정리하고 권장 가이드로 제시
- **PC 환경에서 알고리즘 검증 후 모바일 앱으로 이식**
    - PC 환경에서 카운팅 로직을 먼저 검증한 뒤, 동일한 로직을 Flutter 기반 모바일 앱으로 이식하여 실제 스마트폰 카메라 입력에서도 실시간 카운트가 가능하도록 구현
    - 로그인/회원가입, 저장/기록 조회, 설정 등 부가 기능을 추가하여 개인별 운동 기록 앱 형태로 구현

**기대효과**

- 스마트폰(카메라)만 있다면 운동량(횟수/세트)를 자동으로 기록 가능
- 사용자별 기록 저장을 통해 개인 운동 루틴 관리에 활용 가능

## 2. 시스템 및 기술 구성

### 2.1 전체 구성

### PC 버전(프로토타입/알고리즘 검증)

- 입력: 웹캠 or mp4 영상(OpenCV)
    - 사용 웹캠
        
        
        - RAZER KIYO 웹캠
        
        ![image.png](attachment:05dd7c9a-fa02-4170-974a-3c45c62e71ce:image.png)
        
- 처리: MediaPipe Pose(Python)로 33 랜드마크 추론
- 출력: 스켈레톤 오버레이 + 카운트 상태 표시 + 저장 영상(mp4)

### Mobile 버전(최종 결과물/앱 형태)

- Flutter: 전면 카메라 프리뷰 + UI + 스켈레톤 렌더링 + 카운터 로직 + 저장/기록
- Android(Kotlin): MediaPipe PoseLandmarker 실행(33 관절 추론)
- Channel 통신:
    - MethodChannel: Flutter → Kotlin 프레임 전달
    - EventChannel: Kotlin → Flutter 랜드마크 스트리밍

### 2.2 사용 기술

### 2.2.1 MediaPipe Pose

1. **MediaPipe 란?**
    - MediaPipe는 Goolgle에서 제공하는 AI 기반 실시간 멀티미디어 처리 프레임워크
    - 카메라 영상이나 이미지에서 사람의 얼굴, 손, 몸 자세 등을 실시간으로 분석할 수 있도록 다양한 사전 학습 모델과 처리 도구를 제공함
    - MediaPipe 상세 내용: [MediaPipe (Google)](https://www.notion.so/MediaPipe-Google-2b2d4b8233fa802ca8eec5159c586a51?pvs=21)
2. **MediaPipe Pose 란?**
    - MediaPipe Pose는 MediaPipe에서 제공하는 기능 중 하나로, 영상 속 사람의 전신 자세를 인식하여 33개의 주요 관절 위치를 좌표로 추출해주는 모델
    - 즉, 사람이 화면에서 어떻게 움직이고 있는지를 숫자 좌표 데이터로 바꿔주는 기술
3. **MediaPipe Pose가 하는 일**
    - 카메라 영상이 들어오면 MediaPipe Pose는 아래 작업을 자동으로 수행함
        1. 화면에서 사람을 찾는다.
        2. 사람의 관절 위치를 추정한다.
        3. 각 관절의 좌표(x,y)와 신뢰도(visibility)를 반환한다.
            - 반환하는 값
                - 사람 몸의 주요 관절 위치 33개를 랜드마크로 줌
                    
                    
                    - Pose의 33개 랜드마크
                        - 어깨: LEFT_SHOULDER, RIGHT_SHOULDER
                        - 팔꿈치: LEFT_ELBOW, RIGHT_ELBOW
                        - 손목: LEFT_WRIST, RIGHT_WRIST
                        - 엉덩이: LEFT_HIP, RIGHT_HIP
                        - 무릎: LEFT_KNEE, RIGHT_KNEE
                        - 발목: LEFT_ANKLE, RIGHT_ANKLE
                        - 발끝/뒤꿈치: FOOT_INDEX, HEEL
                        
                    
                    ![image.png](attachment:93c7b4e5-f557-4ae9-8b07-e44225ab0e17:image.png)
                    
                - 각 랜드마크는 다음과 같은 정보를 가짐
                    - x, y: 이미지 기준 정규화 좌표 (0~1)
                        - x = 0.5, y = 0.5 이면 화면 가운데
                        - 픽셀로 바꾸고 싶으면
                            - px = x * frame_width
                            - py = y * frame_height
                    - z: 카메라 기준의 상대 깊이 (스케일 상대값)
                    - visibility: 그 점이 화면에 잘 보이는지 신뢰도
                - MediaPipe Pose 는 사람의 관절 위치(총 33개) 를 두 가지 형태로 제공
                    
                    
                    | 구분 | 의미 | 특징 |
                    | --- | --- | --- |
                    | **2D 좌표** | 화면 기준 위치 | 0~1 사이 값 |
                    | **3D 좌표** | 실제 공간 기준 위치 | 각도 계산 가능 |
                    - 2D 좌표는 화면 기준 위치
                        - x = 0 → 화면 맨 왼쪽
                        - x = 1 → 화면 맨 오른쪽
                        - y = 0 → 화면 맨 위
                        - y = 1 → 화면 맨 아래
                        - 예:`y = 0.53` → 이 관절은 화면 높이의 53% 지점에 있다.
                        
                        ```jsx
                        if results.pose_landmarks:
                        	lm = results.pose_landmarks.landmark
                        ```
                        
                        - 사람이 위로 내려갔다/올라갔다 등의 위치 변화가 중요한 운동에 사용할 수 있음
                    - 3D 좌표는 카메라 기준의 실제 공간 위치
                        - x, y, z 값 제공
                        
                        ```jsx
                        if results.pose_world_landmarks:
                        	lm3d = results.pose_world_landmarks.landmark
                        ```
                        
                        - 위치 변화보다 각도 변화가 중요한 운동 카운트에 사용

### 2.2.2 OpenCV

- OpenCV는 영상과 이미지를 처리하기 위한 대표적인 오픈소스 컴퓨터 비전 라이브러리
- 주요 기능:
    - 카메라 영상 입력
    - 프레임 단위 이미지 처리
    - 좌표 계산 및 시각화
    - 영상 저장 및 디버깅
- 본 프로젝트에서의 역할:
    
    PC 환경에서 
    
    - 웹캠 영상 입력
    - MediaPipe Pose 결과를 화면에 시각화
    - 관절 좌표를 이용한 각도/거리 계산 검증
    - 실험 영상 녹화 및 로그 분석
- OpenCV 상세내용: [OpenCV](https://www.notion.so/OpenCV-2b2d4b8233fa8037a0a9fe12cd012c25?pvs=21)

### 2.2.3 모바일 기술 스택

**Flutter (Dart)**

- Google에서 개발한 크로스 플랫폼 모바일 앱 개발 프레임워크 (Dart 언어 사용)

**Android Kotlin**

- Android 앱 개발에 공식적으로 사용되는 프로그래밍 언어
- MediaPipe Pose는 Flutter 에서 직접 실행하기 어렵고 Android 네이티브 환경에서 실행 가능
- Flutter는 카메라 프레임을 전달하고 Kotlin에서 MediaPipe Pose를 실행한 뒤 결과를 다시 Flutter로 보내는 구조로 설계

**SharedPreferences**

- Android 앱에서 제공하는 간단한 로컬 데이터 저장 방식
    - 키(Key)와 값(Value) 형태로 데이터를 저장
    - 앱 내부에만 저장되며 서버 필요 없음
- 회원가입 정보, 현재 로그인 사용자, 날짜별 운동 기록, 세트별 횟수 및 운동 시간 데이터를 저장하는데 사용됨

### 2.3 시스템 동작 원리

본 프로젝트의 핵심은 MediaPipe가 운동을 카운트 해주는 것이 아니라,

- MediaPipe는 매 프레임마다 관절 위치(점 좌표)를 제공
- 그 좌표 값을 가지고 측정 값을 계산
- 측정 값이 UP/DOWN 같은 상태를 어떻게 바꾸는지를 정의
- DOWN → UP 같은 특정 상태 전환 순간에 count를 +1

하는 방식으로 카운트를 진행한다.

## 3. 개발 진행 과정

### 3.1 초기 접근: 측면 기준 운동량 카운팅

### 3.1.1 프로젝트 기본 설치 및 환경 세팅

[25.12.15 (프로젝트 기본 세팅 / v0.1)](https://www.notion.so/25-12-15-v0-1-2cad4b8233fa803697d8c45efcb2f51f?pvs=21) 

### 소프트웨어 환경

- Python 3.10.8
- MediaPipe (Python) 0.10.21
- OpenCV  4.11.0.86

---

### 1. 가상환경 설치 및 MediaPipe 설치

### 1.1 가상환경 생성

```jsx
python -m venv .venv
```

### 1.2 가상환경 활성화/비활성화

vscode terminal:

```jsx
# 가상환경 활성화
.\.venv\Scripts\activate 

# 가상환경 비활성화
deactivate
```

### 1.3 MediaPipe 설치

```jsx
pip install mediapipe
OR
pip install mediapipe==0.10.21 # 특정 버전 다운로드 가능
```

### 1.4 OpenCV 설치

```jsx
pip install opencv-python
```

---

### 2. 라이브러리 불러오기

```jsx
import cv2  # openCV 라이브러리
import mediapipe as mp  # mediapipe 라이브러리
```

---

### 3. 간단한 사용법

- MediaPipe 기본 패턴:
1. **OpenCV로 카메라 프레임 읽기**
2. BGR → RGB 변환 (`cv2.cvtColor`)
3. MediaPipe에 넣기
4. 결과를 다시 OpenCV로 그리기
5. `imshow`
- MediaPipe 안에 있는 기능 모듈
    - MediaPipe는 기능별 모듈이 `mp.solutions` 안에 들어있음
    
    ```jsx
    mp_hands        = mp.solutions.hands          # 손 추적
    mp_pose         = mp.solutions.pose           # 전신 포즈 추적
    mp_face_mesh    = mp.solutions.face_mesh      # 얼굴 랜드마크(468개 점)
    mp_face_det     = mp.solutions.face_detection # 얼굴 박스 검출
    mp_holistic     = mp.solutions.holistic       # 전체(얼굴+손+포즈) 통합
    mp_selfie_seg   = mp.solutions.selfie_segmentation  # 사람 배경 분리
    ```
    
    | 모듈 이름 (`mp.solutions.XXX`) | 기능 설명 | 주로 쓰는 클래스 |
    | --- | --- | --- |
    | `hands` | 손 21개 랜드마크 추적 | `Hands` |
    | `pose` | 전신 33개 랜드마크 추적 | `Pose` |
    | `face_mesh` | 얼굴 468개 랜드마크 | `FaceMesh` |
    | `face_detection` | 얼굴 위치(박스) 검출 | `FaceDetection` |
    | `holistic` | 얼굴 + 양손 + 포즈 한 번에 | `Holistic` |
    | `selfie_segmentation` | 사람만 분리 (배경 제거) | `SelfieSegmentation` |
- MediaPipe 사용 패턴
    - MediaPipe solutions 모듈은 거의 이 패턴을 따름:
    1. `mp.solutions.XXX` 모듈 핸들 가져오기
    2. `with mp_XXX.ClassName(옵션...) as something:` 으로 객체 생성
    3. 프레임(BGR → RGB 변환) 넣어서 `.process(image)` 호출
    4. `results.~~~` 에서 결과 꺼내 쓰기
    - 예시
        
        ```python
        import cv2
        import mediapipe as mp
        
        mp_hands = mp.solutions.hands  # 1) 모듈 불러오기
        mp_draw  = mp.solutions.drawing_utils  # 그리기 유틸
        
        cap = cv2.VideoCapture(0)  # 카메라 열기 (0번)
        
        # 2) Hands 객체 만들기
        with mp_hands.Hands(
            max_num_hands=2,           # 최대 손 개수
            min_detection_confidence=0.5,  # 검출 신뢰도
            min_tracking_confidence=0.5    # 추적 신뢰도
        ) as hands:
            while True:
                success, frame = cap.read()
                if not success:
                    break
        
                # 3) BGR → RGB 변환 (OpenCV는 BGR, MediaPipe는 RGB 사용)
                img_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        
                # 4) MediaPipe에 넣고 처리
                result = hands.process(img_rgb)
        
                # 5) 결과(랜드마크)가 있으면 그림 그리기
                if result.multi_hand_landmarks:
                    for handLms in result.multi_hand_landmarks:
                        mp_draw.draw_landmarks(
                            frame, handLms, mp_hands.HAND_CONNECTIONS
                        )
        
                cv2.imshow("Hands", frame)
                if cv2.waitKey(1) & 0xFF == ord('q'):
                    break
        
        cap.release()
        cv2.destroyAllWindows()
        ```
        
- OpenCV 자주 쓰는 핵심 기능
    - 이미지 입/출력
    
    | 기능 | 코드 | 설명 |
    | --- | --- | --- |
    | 이미지 읽기 | `cv2.imread("img.png")` | 파일을 배열로 읽음 |
    | 이미지 저장 | `cv2.imwrite("save.png", img)` | 파일로 저장 |
    - 화면 출력
    
    | 기능 | 코드 | 설명 |
    | --- | --- | --- |
    | 이미지 표시 | `cv2.imshow("title", img)` | 창으로 보여줌 |
    | 키 입력 대기 | `cv2.waitKey(0)` | 아무 키 입력까지 대기 |
    | 창 닫기 | `cv2.destroyAllWindows()` | 모든 창 닫기 |
    - 카메라 사용
    
    | 기능 | 코드 | 설명 |
    | --- | --- | --- |
    | 카메라 열기 | `cv2.VideoCapture(0)` | 0번 웹캠 |
    | 영상 읽기 | `ret, frame = cap.read()` | ret=True면 성공 |
    | 카메라 종료 | `cap.release()` | 자원 해제 |
    - 색 공간 변환
    
    | 기능 | 코드 | 설명 |
    | --- | --- | --- |
    | BGR → RGB | `cv2.cvtColor(img, cv2.COLOR_BGR2RGB)` | MediaPipe에서 필수 |
    | BGR → Gray | `cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)` | 얼굴 검출 등에 활용 |
    - 도형 그리기
    
    | 기능 | 코드 | 설명 |
    | --- | --- | --- |
    | 점 | `cv2.circle(img, (x,y), 3, (0,255,0), -1)` | (-1 = 채움) |
    | 선 | `cv2.line(img, p1, p2, (255,0,0), 2)` | 두께 2 |
    | 사각형 | `cv2.rectangle(img, (x1,y1),(x2,y2),(0,0,255),2)` |  |
    - 텍스트 그리기
    
    ```jsx
    cv2.putText(img, "Hello", (50,50),
    cv2.FONT_HERSHEY_SIMPLEX, 1,
    (255,255,255), 2)
    ```
    
    - 이미지 크기 조절
    
    ```jsx
    resized = cv2.resize(img, (640, 480))
    ```
    
    - 이미지 회전/반전
    
    ```jsx
    flip = cv2.flip(img, 1)        # 좌우 반전
    rotate = cv2.rotate(img, cv2.ROTATE_90_CLOCKWISE)
    ```
    
    - ROI (자르기)
    
    ```jsx
    # 배열 슬라이싱 규칙: [y1:y2, x1:x2]
    crop = img[100:300, 200:500]
    ```
    

### 3.1.2 측면 기준 카운팅 구조 설계 및 검증

[25.12.16 (v0.2 측면 기준 운동 카운팅)](https://www.notion.so/25-12-16-v0-2-2cbd4b8233fa80be9007c1ccdfd58391?pvs=21) 

- Mixamo + Blender 기반의 가상 운동 영상을 활용하여 측면 촬영 기준 스쿼트, 푸쉬업, 크런치 세 가지 운동 카운팅 알고리즘의 동작을 설계하고 검증함
    - Mixamo 사용법
        
        ### 1. Mixamo 페이지 접속 & 로그인
        
        - Mixamo 페이지 접속 : https://www.mixamo.com/#/
        - 구글 아이디로 로그인
        
        ### 2. 애니메이션 적용 및 다운로드
        
        1. Browse Animations 클릭
        
        ![Section 2 (18).png](attachment:88c9dfe5-463d-45ac-b154-02f63df076ce:ba3f4b4d-b4cd-47e2-912d-cab0f7e30fce.png)
        
        1. 원하는 동작 검색
        
        ![Section 2 (19).png](attachment:ee96f0f1-5105-4cf9-a051-d4519e27a599:c27e1c95-3e7e-48f4-b395-599f749020fc.png)
        
        1. DOWNLOAD 클릭 
        2. Format → FBX Binary 
        (다른 설정은 기본 값 유지)
        3. DOWNLOAD
        
        ![Section 2 (20).png](attachment:b5d1fbc7-98fe-46a8-82a1-23b4f32dd513:bd61e860-f5be-4f2b-9188-8455b22c6ab2.png)
        
        1. .fbx 로 다운로드 되었는지 확인
        
        ![image.png](attachment:f8cc4c17-233d-497f-b1a7-778cb9afdf42:image.png)
        
    - Blender 사용법
        - Blender 다운로드: https://www.blender.org/download/
        
        ![Section 2 (22).png](attachment:16fac5bd-4b91-4d45-9276-320daf8899d5:9c38ef57-c846-422a-946a-a4acde23b45f.png)
        
        ![image.png](attachment:c337a272-6ccc-495f-a21b-5bd69c0488ff:image.png)
        
        ### 2. Blender 기본 사용법
        
        - 마우스 조작
            - 휠 클릭 + 드래그 → 회전
            - 휠 스크롤 → 줌 인/아웃
            - Shift + 휠 클릭 + 드래그 → 평행 이동
        - 선택/이동/회전/스케일
            - 선택: 마우스 왼쪽 클릭
            - 이동: G
            - 회전: R
            - 크기 변경: S
        - 축 고정
            - X축: G → X
            - Y축: G → Y
            - Z축: G → Z
        - 3D 뷰 변환
            
            
            | Numpad 키 | 기능 |
            | --- | --- |
            | **1** | 정면(Front) |
            | **Ctrl + 1** | 뒷면(Back) |
            | **3** | 오른쪽(Right) |
            | **Ctrl + 3** | 왼쪽(Left) |
            | **7** | 위(Top) |
            | **Ctrl + 7** | 아래(Bottom) |
            | **0** | 카메라 뷰(Camera View) |
        
        ### 3. Blender에서 Mixamo FBX 가져오기
        
        1. 파일 → 가져오기 → FBX 선택 
        
        ![Section 2 (23).png](attachment:6faad918-7964-4aa0-a154-e13276818cb6:4dec7738-ae64-476e-95b5-3f52f8e2c59e.png)
        
        1. 다운받은 Mixamo 파일 선택 → FBX를 가져오기 선택
        
        ![Section 2 (24).png](attachment:02c49383-a24c-443f-97a7-ebf63fa8b763:491bf86f-cc67-4cf2-96d5-dba1d48e97f7.png)
        
        ![image.png](attachment:c4c3b69b-80f0-4504-957f-216cab634d0f:image.png)
        
        ### 4. 모델 위치/크기 조정
        
        1. 모델 선택
        2. 3D 뷰 변환 (Numpad로 조정)
        3. 마우스 휠과 단축키로 모델 위치/크기 조정
        
        ![image.png](attachment:24549263-e93b-42dc-bf19-8d1a9588a597:image.png)
        
        <aside>
        🛠
        
        예시) 
        
        1. 모델 선택
        2. Numpad1 로 정면 보게 하기
        3. 단축키 G → Z로 모델 아래로 이동한 후 Enter
        </aside>
        
        ### 5. 카메라 세팅
        
        - 원하는 구도로 만드는 작업
        1. 지금 보고 있는 화면을 카메라로 정하기
        
        ![image.png](attachment:e06b6635-663c-4e72-a54a-e231b17bd290:image.png)
        
        ```jsx
        # 현재 시점이 카메라 시점으로 고정
        Ctrl + Alt + Numpad 0
        ```
        
        1. 카메라만 따로 이동
        
        - 오른쪽 탭의 씬 컬렉션에서 카메라 선택 후:
            - 이동: G
            - Z 축 이동: G → Z
            - 회전: R
            - 상하 회전: R → X
            - 좌우 회전: R → Z
        
        ![image.png](attachment:68d60f3b-94d4-4695-91a3-1acbc30212df:image.png)
        
        ### 6. 애니메이션 재생 테스트
        
        - 재생: Space
        - 시작, 종료 시점 설정 가능
        
        ![Section 2 (25).png](attachment:f864dfd3-301e-4165-913d-156a30c23e7b:b53cde34-c875-41ed-afc1-1bac4d9933eb.png)
        
        ### 7. 최종 렌더링 (영상 출력)
        
        - 오른쪽 Properties → Output (출력 프로퍼티스)
        
        1. 출력 형식 설정
            - 원하는 해상도 설정
            - 1280x720ㄴ
        2. 프레임 범위 설정
            - 프레임 시작/종료 범위 설정
            
        
        ![Section 2 (26).png](attachment:fd09aec4-4670-4c1b-9c9b-cccf81b64cfb:a91ea91e-e878-4ed5-8cf3-4a1d72594e3b.png)
        
        1. 출력 폴더 설정
        2. 파일 형식 → 비디오 선택
        
        ![Section 3 (5).png](attachment:d819ba04-74d7-43ba-85b8-1676cf703bb5:46a548f4-1f6e-45a2-a638-5db43482268e.png)
        
        1. Ctrl + F12 로 애니메이션 전체 렌더링
            - F12 는 현재 시점 카메라 기준으로 한 장의 이미지 렌더링
- 이후 실제 웹캠 환경에서 최종 튜닝을 수행하는 것을 목표로 함

### 스쿼트

https://github.com/pnkmem432/workout-counter-pc/blob/v0.2/v0.2/main_squat.py

### **기본 스쿼트 자세 단계**

1. **준비 자세:**
    - 발은 어깨너비로 벌리고, 발끝은 10~30도 정도 바깥쪽으로 향하게 합니다.
    - 팔은 앞으로 뻗거나 가슴 앞에 모아 균형을 잡습니다.
    - 허리는 펴고 복부에 힘을 주어 긴장시킵니다.
2. **내려가는 동작:**
    - 숨을 들이마시면서 엉덩이를 뒤로 빼고, 무릎이 접히는 동시에 골반도 같이 접는 느낌으로 천천히 앉습니다.
    - 마치 의자에 앉는 것처럼, 무릎은 발가락 방향(두 번째 발가락)으로 벌리며 내려갑니다.
    - 상체는 너무 앞으로 숙이지 않고, 허리가 굽지 않도록 자연스럽게 유지합니다.
    - 깊이는 무릎이 발끝을 넘지 않는 범위나, 엉덩이가 무릎 높이까지 닿는 정도까지 내려갑니다.
3. **올라오는 동작:**
    - 발바닥 전체로 바닥을 밀어준다는 느낌으로 힘을 주며 일어섭니다.
    - 일어설 때 무릎을 완전히 펴지 않고 살짝 굽힌 상태에서 멈추고, 엉덩이와 허벅지에 힘을 느끼며 괄약근을 조여줍니다.
    - 숨을 내쉬면서 올라옵니다.

![image.png](attachment:570e408a-36a9-4dfe-a194-363e8398e09f:image.png)

![image.png](attachment:954158d1-0442-4bc1-8b26-d10ad0c5cf3f:image.png)

### 스쿼트 동작 정의

**판정 기준**

- **주요 지표**
    - 무릎 각도: hip–knee–ankle 각도
- **보조 고려 사항**
    - 좌·우 중 랜드마크 가시성(visibility)이 높은 쪽을 사용
    - EMA(지수 이동 평균)를 적용하여 각도 변화의 노이즈를 완화

**동작 상태 정의**

- **DOWN(하강/이완)**
    - 무릎이 굽혀져 hip–knee–ankle 각도가 감소한 상태
    - 약 90도 이하
- **UP(상승/수축)**
    - 무릎이 펴지며 hip–knee–ankle 각도가 증가한 상태
    - 약 150도 이상

**카운트 로직**

- DOWN 상태에서 충분히 하강한 후
- 다시 UP 상태로 완전히 복귀하면 **1회로 판정**

### Count 기준 값

- **hip(엉덩이)–knee(무릎)–ankle(발목) 내부각  →** `angle_3pts(a, b, c)`
    - **A = 엉덩이(hip), B = 무릎(knee), C = 발목(ankle)**
    - **B(무릎)를 꼭짓점**
    - 다리 **완전 쭉 펴면**: 엉덩이–무릎–발목이 거의 일직선 → **각도 ≈ 170~180도**
    - 무릎을 **많이 굽히면(스쿼트 내려감)**: 꺾이니까 → **각도 작아짐 (예: 60~100도)**
- 서 있음(UP_TH) = 150
- 앉음(DOWN_TH) = 90

![Section 2 (15).png](attachment:ea30a198-d15d-437b-8fa5-43b26a202159:fcc5470f-ee7d-42f9-a768-829f1ef57d30.png)

![Section 2 (14).png](attachment:d2609196-e679-46fa-8870-979d8692c817:47d36762-5533-4ce2-adb8-fee8d03e05b4.png)

[KakaoTalk_20251215_153146247.mp4](attachment:83547933-e9dc-431c-b401-1d0aff1333d1:KakaoTalk_20251215_153146247.mp4)

### 참고 문헌

### Knee biomechanics of the dynamic squat exercise — 동적 무릎 생체역학 스쿼트 운동

[ARTIGO-AGACHAMENTO-02.pdf](attachment:c02ecfd7-4d86-4194-b676-e934821b2766:ARTIGO-AGACHAMENTO-02.pdf)

- 동적 스쿼트 동작 중 무릎 관절에 가해지는 힘(전단력 및 압축력), 무릎 주변 근육 활동, 그리고 무릎 안정성을 검토하는 것을 목적으로 하는 논문
- 스쿼트를 분석할 때 **무릎 관절 각도 를** 중심 변수로 사용
    - 무릎 굽힘 각도는 대퇴 - 경골 사이 각도 (엉덩이-무릎-발목)
- 카운팅 기준 각도
    - **스쿼트 1회 카운트 기준:** 0도(시작) → 최소 기준 각도 (하강) → 0도(종료)의 연속된 동작이 완료되었을 때 1회로 카운트
        
        
        | **구분** | **각도 기준 
        (무릎 굴곡 각도)** | **동작 기준** |
        | --- | --- | --- |
        | **시작/종료 지점** | 0도 | 무릎과 엉덩이를 완전히 편 직립 자세 |
        | **일반/표준 스쿼트** | 약 100도 | 허벅지가 지면과 평행한 지점 |
        | **재활/초보자 스쿼트** | 약 50도 | 무릎 관절에 가해지는 힘을 최소화하는 기능적 범위 |
        | **딥 스쿼트** | 100도 이상 
        (최대한 깊이) | 건강한 사람에게도 인대/반월판 손상 위험 증가로 비권장됨 |

### 스쿼트 동작 시 웨이트 벨트 착용 전·후에 따른 운동역학적 분석

[Sports Biomechanical Analysis before and after Applying Weight Belt during Squat Exercise.pdf](attachment:770909a9-bf12-4ba9-90b3-64619463e799:Sports_Biomechanical_Analysis_before_and_after_Applying_Weight_Belt_during_Squat_Exercise.pdf)

- 스쿼트 분석 시 ankle, knee, hip 관절의 기하학적 관계를 이용함

### 구글

![image.png](attachment:7cfc6475-ee15-4d3e-9be6-a403e6c2d42f:image.png)

### 푸쉬업

https://github.com/pnkmem432/workout-counter-pc/blob/v0.2/v0.2/main_pushup.py

### 기본 푸쉬업 자세 단계

**기본 준비 자세** 

1. **엎드리기**: 바닥에 엎드려 손바닥을 어깨너비보다 약간 넓게 벌려 바닥을 짚습니다.
2. **몸 정렬**: 발끝으로 몸을 지지하고, 엉덩이, 복근, 허벅지에 힘을 주어 몸 전체를 머리부터 발끝까지 일직선으로 만듭니다.
3. **시선**: 시선은 바닥을 향해 약간 앞쪽을 봅니다.

**동작 방법**

1. **내려가기**: 숨을 들이마시며 팔꿈치를 몸통에서 약 45도 각도로 벌리며 가슴이 바닥에 닿기 직전까지 천천히 내려갑니다.
2. **가슴 펴기**: 내려갈 때 가슴을 활짝 펴고, 팔을 살짝 바깥쪽으로 돌려 어깨 안정성을 확보합니다.
3. **올라오기**: 숨을 내쉬며 가슴과 팔 근육에 힘을 주어 시작 자세로 돌아옵니다.

![image.png](attachment:51d1ffc2-c978-4f72-87d2-df06fee52ab6:image.png)

### 푸쉬업 동작 정의

**판정 기준**

- **주요 지표**
    - 팔꿈치 각도: shoulder–elbow–wrist 각도
- **보조 고려 사항**
    - 팔꿈치, 어깨, 손목 랜드마크의 가시성
    - EMA를 적용하여 순간적인 관절 인식 오류를 완화

**동작 상태 정의**

- **DOWN(하강/이완)**
    - 팔꿈치가 굽혀져 shoulder–elbow–wrist 각도가 감소한 상태
    - 약 90도 이하
- **UP(상승/수축)**
    - 팔꿈치가 펴지며 shoulder–elbow–wrist 각도가 증가한 상태
    - 약 160도 이상

**카운트 로직**

- DOWN 상태에서 충분히 내려간 후
- 다시 UP 상태로 완전히 올라오면 **1회로 판정**

### Count 기준 값

- shoulder(어깨) - elbow(팔꿈치) - wrist(손목) → 팔꿈치 각도
    - A = 어깨(shoulder), B = 팔꿈치(elbow), C = 손목(wrist)
    - B(팔꿈치)를 꼭짓점
    - 팔을 쭉 펴면: 거의 일직선 → 각도 ≈ 160~180°
    - 팔을 굽히면 (내려감): V자 형태 → 각도 작아짐 (예: 60도~100도)
- 올라옴(UP_TH) = 160
- 내려감(DOWN_TH) = 90
- 로직 예시
    - 시작(UP): 175
    - 내려감: 150 → 120 → 95 → 85  (90 아래로 내려감 → DOWN)
    - 올라감: 100 → 135 → 165 (160 위로 올라옴 → UP & count+1)

![Section 2 (16).png](attachment:f935bc56-7eda-4f9c-91fd-8445b5c8a671:064906e9-9f03-4db1-827e-92d279d8c2b6.png)

![Section 3 (3).png](attachment:f763c8af-c58f-4945-8e99-d866c020f0e0:6d6445f1-56f7-4719-905f-250718bd94cd.png)

[KakaoTalk_20251215_163401215.mp4](attachment:7a30ee75-364b-4499-ba8d-1bb3428af97c:KakaoTalk_20251215_163401215.mp4)

### 참고 문헌

### 팔 굽혀 펴기에 대한 생체역학 분석

[Sports Biomechanical Analysis before and after Applying Weight Belt during Squat Exercise.pdf](attachment:32c1c60f-51e9-410f-b7ce-8ba5cb3b752c:Sports_Biomechanical_Analysis_before_and_after_Applying_Weight_Belt_during_Squat_Exercise.pdf)

- 푸쉬업 동작 분석에서 팔꿈치 관절각은 상완과 전완이 이루는 각도로 정의
    - 상완: 어깨 - 팔꿈치
    - 전완: 팔꿈치 - 손목
- 팔꿈치 관절각의 변화에 따라 푸쉬업의 상,하 국면을 구분
    - 상하운동 시 (최대 각 = θ + 90°로 운동) θ < 90°, θ = 90°, θ > 90° 구간에 따라 동작 특성이 달라진다.
    - 90도 전후를 동작 특성이 변화하는 주요 기준 각도로 활용함

### 푸쉬업 자세 시 팔굽관절 각도와 어깨넓이 간격에 따른 4구획별 배 곧은근 근활성도 비교

[푸시 업 자세 시 팔굽관절 각도와 어깨넓이 간격에 따른 구획별 4 배 곧은근 근활성도 비교.pdf](attachment:ba146c46-aee0-49f7-9143-f52b06c2a3c9:푸시_업_자세_시_팔굽관절_각도와_어깨넓이_간격에_따른_구획별_4_배_곧은근_근활성도_비교.pdf)

- 푸쉬업 수행 시 팔굽관절 각도를 0도, 45도, 90도 로 구분하여 분석하여 팔꿈치 굴곡 각도에 따라 근활성도와 운동 특성이 유의미하게 달라짐을 보고함
- 0도가 완료된 자세의 기준, 90도가 근육 활성, 부하 특성이 분명히 달라지는 운동 상태 임을 보임

### 구글

![image.png](attachment:078e9928-dc14-4454-a740-24babb081bff:image.png)

### 크런치

https://github.com/pnkmem432/workout-counter-pc/blob/v0.2/v0.2/main_crunch.py

### 기본 크런치 자세 단계

**준비 자세**

- **매트 눕기:** 등을 대고 편안하게 눕습니다.
- **다리:** 무릎을 90도로 굽혀 세우고 발바닥을 바닥에 댑니다.
- **손:** 깍지를 끼지 않고 양손을 귀 옆이나 머리 뒤에 가볍게 댑니다.
- **허리:** 허리 아치(요추 커브)가 살짝 있는 상태에서 복부에 힘을 주어 허리를 바닥에 밀착시킵니다.

**동작**

1. **숨을 들이마시고**, 상체를 웅크리면서 **숨을 뱉습니다.**
2. **어깨 윗부분과 등 윗부분만** 바닥에서 살짝 들어 올립니다. (윗몸일으키기처럼 많이 들지 않음)
3. **턱은 가슴 쪽으로 당기고**, 복근을 최대한 수축하며 잠시 멈춥니다.
4. **숨을 들이마시면서** 천천히 시작 자세로 돌아갑니다. 이때 복근의 긴장이 완전히 풀리지 않도록 합니다.

![image.png](attachment:85208338-8690-43ef-af29-7cdb29d3a553:image.png)

![image.png](attachment:e6cb7377-a589-4e7f-bdd7-8b5972119e33:image.png)

### 크런치 동작 정의

**판정 기준**

- **주요 지표**
    - **상체 기울기(TILT)**
        - hip–shoulder 선이 수평에 대해 이루는 기울기 각도
        - 상체가 세워질수록 값이 증가
    - **어깨 들림 높이(DY)**
        - hip을 기준으로 한 shoulder의 상대적인 y좌표 변화량
        - 어깨가 바닥에서 들릴수록 값이 증가
- **보조 고려 사항**
    - 두 지표 모두 EMA를 적용하여 안정화
    - 랜드마크 가시성이 높은 좌·우 측을 선택

**동작 상태 정의**

- **DOWN(이완/누운 상태)**
    - 상체 기울기가 작고, 어깨 들림 높이가 낮은 상태
    - TILT < **22° /** DY < **90 px**
- **UP(수축/말림 상태)**
    - 상체 기울기가 커지고, 어깨가 힙 기준으로 충분히 들린 상태
    - TILT > **30° /** DY > **120 px**

**카운트 로직**

- DOWN 상태에서
- **상체 기울기와 어깨 들림 높이가 동시에 임계값을 초과하여 UP 상태에 도달한 후**
- 다시 DOWN 상태로 완전히 복귀하면 **1회로 판정**
- 크런치의 경우 단일 각도만으로는 목 들림, 골반 말림 등의 오인식이 발생할 수 있으므로, 상체 기울기와 어깨 들림 높이를 함께 사용하여 동작의 안정성과 정확도 향상

### Count 기준 값

- TILT = 상체 기울기 각도
    - hip(엉덩이) → shoulder(어깨)를 잇는 선이 수평선(바닥)에 대해 얼마나 기울어졌는지(0~90°)
    - 계산 개념
        - hip→shoulder 선이 **가로에 가까우면**(누움) → `TILT` 작음
        - hip→shoulder 선이 **대각선/세워지면**(올라옴) → `TILT` 큼
- DY = 어깨 들림 높이 (hip 기준)
    - `DY = hip_y - shoulder_y` (픽셀 단위)
    - 계산 개념
        - **어깨가 위로 올라가면** shoulder_y 값이 작아짐 → `DY`가 커짐
        - **다시 누우면** shoulder_y 값이 커짐 → `DY`가 작아짐
            - 위로 올라갈수록 y 값은 작아지고 아래로 내려갈수록 y 값은 커짐 (openCV, MediaPipe 로직)
    
    ```
    TILT_UP_TH   = 30.0   # 이보다 크면(상체가 충분히 세워짐) -> UP 후보
    TILT_DOWN_TH = 22.0   # 이보다 작으면(상체가 다시 누움) -> DOWN 후보
    ```
    
    ```python
    DY_UP_TH     = 120.0   # delta_y가 이보다 크면(어깨가 힙보다 충분히 위) -> UP
    DY_DOWN_TH   = 90.0   # delta_y가 이보다 작으면(어깨가 내려감) -> DOWN
    ```
    
- 로직
    - **TILT(상체가 세워졌는지)** + **DY(어깨가 실제로 들렸는지)** 두 조건이 동시에 충족될 때만 상태 전환
        - 허리만 말리는 경우 → TILT만 증가
        - 목만 드는 경우 → DY만 증가
        - 이 외 반동, 흔들림 등의 요인으로 오인식의 경우를 최소화 하기 위함
    - **충분히 올라와서(UP 조건 만족) → 다시 충분히 내려와야(DOWN 조건 만족) 1회로 카운트**

![Section 2 (17).png](attachment:e0ba441a-35c7-469f-8b34-509321b58587:2a8f770e-7a30-4796-83db-77d3c5ea0e5b.png)

![Section 3 (4).png](attachment:19e53199-a214-4116-85ae-f1ba75f96525:f1d3ee48-7e0d-46bd-8ad9-0f71b996393e.png)

### 참고 문헌

### 복부 크런치는 안전하고 효과적인 운동인가 — Abdominal Crunches Are/Are Not a Safe and Effective Exercise

[Abdominalcrunchesare-arenotasafeandeffectiveexercise.pdf](attachment:b5c84228-7b21-467c-a4c1-617fca41412e:Abdominalcrunchesare-arenotasafeandeffectiveexercise.pdf)

- 크런치의 본질적 동작 = 척추/상체 굴곡
    - 상체 말림(기울기, 어깨 들림) 을 기준으로 보는 것으로 정의한 근거
- 크런치는 반복적으로 다음 구조로 설명됨 :
    - Flexion phase (상체 굴곡, 복근 수축)
    - Extension / return phase (이완, 원위치)
    - 굴곡 상태까지 갔다가 다시 이완으로 돌아오면 1회로 카운트 로직에 대한 근거
- 크런치는 관절 하나가 아니라 몸통 전체의 움직임
    - 각도 하나로 정의하기 어렵고 상체 전체의 움직임을 보아야 함
        - hip-shoulder 기울기
        - 어꺠 절대 높이 변화

### 구글 —https://korean.mercola.com/sites/articles/archive/2021/11/19/%ED%81%AC%EB%9F%B0%EC%B9%98.aspx

![image.png](attachment:f11c9377-598c-4ac6-92e0-64412d963458:image.png)

- 어깨 위치가 얼마나 위로 이동했는가
    - 하지만 절대적인 cm는 카메라-사람 거리나 카메라 화각, 영상 해상도, 사람 체형, 카메라 높이/각도 등에 따라 달라질 수 있으므로 hip을 기준으로 한 shoulder의 상대적인 y좌표 변화량을 사용하여 어깨 들림을 판단

### 3.2 방향 전환: 정면 기준 + 상태 머신 개념 도입

[25.12.22 (v0.3 정면 기준 운동 카운트)](https://www.notion.so/25-12-22-v0-3-2d1d4b8233fa806a877dcf0b5f29fcbc?pvs=21) 

- 측면 촬영 기준(각도 기반)으로 스쿼트/푸쉬업/크런치 카운팅은 비교적 안정적으로 정의할 수 있었지만 실제 사용 환경을 고려하면 사용자가 정면에 휴대폰을 두고 운동하는 형태가 더 자연스럽고 사용자 입장에서 더 편리하기 때문에 본 프로젝트는 실제 사용 가능한 시스템을 목표로 정면 기준에서도 카운팅이 가능하도록 로직을 재설계 하였다.
- 정면 촬영에서는 측면 촬영과 달리 다음과 같은 문제가 자주 발생한다.
    - 팔/몸통 등이 겹치면서 관절이 가려짐
    - 무릎/팔꿈치 등이 카메라에 대해 접히는 방향이 잘 보이지 않아 각도 변화가 작거나 왜곡될 수 있음
    
    즉, ‘무릎 각도 90도 이하’ 같은 절대 각도 기준은 안정적이지 않음
    
- 카운트 1회를 절대 각도 기준이 아닌 위치 변화에 따른 상태의 순서 변화로 정의

### 3.2.1 정면 기준 카운팅 로직 설계

### 공통 로직

### 1) 프레임 처리

1. 영상에서 한 프레임 읽기
2. 크기 조절 (resize)
3. BGR → RGB 변환
4. MediaPipe로 관절 점 추정
5. 관절 점을 이용하여 운동 메트릭 값 (숫자) 계산
6. 메트릭 값을 이용하여 상태(state) 판단
7. 상태에 따른 count 증가
8. 화면에 글씨/스켈레톤 표시 + 영상 저장

### 2) `visibilty` 체크

- 관절이 잘 안 잡히는 프레임에서 계산하면 오류가 커짐
- 여러 관절의 `visibilty` 를 최솟값으로 묶어서 `VIS_TH` 이상일 때만 계산
    - ex)
    
    ```jsx
    vis_used = min(
    L_SH.visibility,
    R_SH.visibility,
    L_HIP.visibility,
    R_HIP.visibility
    )
    
    if vis_used >= VIS_TH:
        계산 진행
    ```
    

### 3) EMA(지수 이동  평균)로 값 안정화

- 프레임마다 측정되는 값은 흔들리기 쉬워서 부드럽게 만들기 위해 EMA를 사용

```jsx
ema = (1-alpha) * prev + alpha * new
```

- `ALPHA`가 작을수록 → 더 부드럽지만 반응 느림
- `ALPHA`가 클수록 → 더 빠르지만 튐

### 4) 상태 머신

- UP: 올라온 상태
- DOWN: 내려간 상태
- DOWN → UP 으로 복귀할 때 `count + 1`
- 모든 운동의 임계값은 실제 운동 환경에 맞춰 다시 재세팅 할 필요가 있음

### 스쿼트

- 정면에서 스쿼트 1회를 힙이 내려갔다가 다시 올라오는 동작으로 카운트
    - 정면 스쿼트는 무릎 각도보다 사람 전체가 아래로 내려갔다가 올라오는 움직임이 더 잘 보임
    - 힙 중앙(y)이 얼마나 내려갔는지를 측정

**사용 관절**

- LEFT_HIP, RIGHT_HIP
- LEFT_SHOULDER, RIGHT_SHOULDER

**로직**

- **baseline**: 사람마다 키, 카메라 거리, 시작 위치가 다르기 때문에 시작 초반 `BASELINE_FRAMES=10` 프레임 동안 힙 중앙 y값을 평균내서 기준 `hip0_y` 로 저장
- 힙 중앙 좌표: `mid_hip_y = (L_HIP.y + R_HIP.y)/2`
- 기준 대비 내려감: `d_hip = mid_hip_y - hip0_y` (y는 아래로 갈수록 커지므로 내려갈수록 +)
- 사람 크기 보정(정규화):
    - `torso_len = distance(mid_shoulder, mid_hip)`
    - `d_used = d_hip / torso_len` → 힙이 내려간 정도를 내 몸통 길이로 나눈 값
- `D_DOWN_TH=0.18` 넘으면 DOWN
- `D_UP_TH=0.08` 아래면 UP + count++
- **생성 영상 실험**
    
    https://drive.google.com/drive/u/0/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo
    
    https://github.com/pnkmem432/workout-counter-pc/tree/v0.3/v0.3
    
    ![image.png](attachment:bf1f69c4-3393-478c-b624-9f78850477a1:18734e8e-327a-40c3-98e8-cedbd16d1e22.png)
    
    ![image.png](attachment:5baea309-98da-4b57-94bd-6e87850764d7:60bea057-9da7-4ff4-8381-e670ec8a55f3.png)
    
    - 화면 오버레이
        - state: 상태 (UP/DOWN)
        - count: 카운트 횟수
        - D_HIP_NORM(EMA): 지금 힙이 baseline(서있는 기준) 대비 얼마나 내려갔는지 숫자로 보여줌
            - 현재 힙 중앙 y: `mid_hip_y`
            - baseline 힙 y: `hip0_y`
            - 내려간 정도: `d_hip = mid_hip_y - hip0_y` (y축은 아래로 내려갈수록 커짐)
            - 사람 크기 보정(정규화): `d_used = d_hip / torso_len`
            - 값이  커지면 힙이 더 내려감, 0에 가까우면 baseline 위치랑 비슷함
        - MIN/MAX: 지금까지 관측된 D_HIP_NORM의 최솟값/최댓값
        - TH(D/U): 임계값 2개
            - D = D_DOWN_TH = 0.18
                - UP → DOWN 전환 기준 (힙이 이 정도 이상 내려가야 내려갔다 인정)
            - U = D_UP_TH = 0.08
                - DOWN → UP 전환 기준 (힙이 이 정도 이상 올라와야 올라왔다 인정)
        - Base_HIP_Y: baseline(서 있는 구조)에서의 힙 중앙 y 값
            - `BASELINE_FRAMES=10` 프레임 동안 `mid_hip_y`를 평균내서 만든 값
            - 0.535는 화면 높이의 53.5% 지점에 힙이 위치한다는 의미
        - BASE_TORSO: baseline 에서의 torso 길이(어깨 중앙 ↔ 힙중앙 거리) 값
            - baseline 자세에서 어깨 중앙과 힙 중앙 사이의 거리가 화면 높이를 1로 보았을 때 약 18.4% 정도다 라는 뜻
        - VIS(min4): 신뢰도 점수
            - min4: LEFT_HIP, RIGHT_HIP, LEFT_SHOULDER, RIGHT_SHOULDER 4개 관절 visibility 중 최솟값
            - 0~1 범위
    - 스쿼트는 정면에서 보면 무릎이 얼마나 굽혀졌는지 보다 사람이 전체적으로 아래로 내려갔다가 다시 올라오는 움직임이 가장 잘 보임
        - **엉덩이 위치가 기준보다 충분히 내려갔다가 다시 올라오는 순간을 스쿼트 1회로 카운트**
    - 힙이 내려간 정도를 내 몸통 길이로 나눠서 키가 작든 크든 카메라가 가깝든 멀든 자기 몸 기준으로 얼마나 내려갔는지를 계산

### 푸쉬업

- 정면에서 푸쉬업 1회를 어깨가 내려갔다가 다시 올라오는 동작으로 카운트

**사용 관절**

- LEFT_SHOULDER, RIGHT_SHOULDER

**로직**

- **baseline:** UP 자세를 찾아 시작
    - DOWN 에서 올라오면서 시작할수도 있고, UP 상태에서 시작할수도 있다고 생각함
    - UP 자동 탐지 로직:
        - `sh_min_y`: 지금까지 본 어깨 중 가장 작은 값
        - 현재 프레임이 `mid_sh_y <= sh_min_y + UP_WINDOW_MARGIN` 이면 UP 후보
        - UP 후보가 `UP_STABLE_FRAMES=5` 이상 연속이면 baseline 누적 시작
        - 누적이 `BASELINE_FRAMES=10` 되면 baseline 확정(`sh0_y`)
- 어깨 중앙: `mid_sh_y`
- baseline 대비 내려감: `d_sh = mid_sh_y - sh0_y`
- 정규화
    - 내 몸 기준으로 얼마나 내려갔는지
    - `d_used = d_sh / shoulder_width` → 어깨가 내려간 거리 / 내 어깨 폭
        - 어깨 폭은 푸쉬업 동작 중에 거의 변하지 않고 카메라 거리 변화에도 비례해서 같이 변함
        - ex) 어깨 폭이 0.2, 어깨가 0.04 내려갔다면 내 몸 기준으로 20% 내려갔다고 판단
- `D_DOWN_TH=0.25` 넘으면 DOWN
- `D_UP_TH=0.10` 아래면 UP + count++
- **생성 영상 실험**
    
    https://drive.google.com/drive/u/0/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo
    
    https://github.com/pnkmem432/workout-counter-pc/tree/v0.3/v0.3
    
    ![image.png](attachment:10ffb902-23cc-4440-a94a-19216601177a:be3c97c9-8185-4c55-bd81-fdd846bea4d5.png)
    
    ![image.png](attachment:8a321102-89d6-4ed7-a0b6-6be791a2fcd0:906c2b45-3ffc-420a-8c55-e19035feede3.png)
    
    - 화면 오버레이
        - state: 상태 (UP/DOWN)
        - count: 카운트 횟수
        - D_SH_NORM(EMA): baseline(UP 자세)의 어깨 높이(sh0_y) 에 비해 지금 어깨가 얼마나 
        내려갔는지/올라갔는지를 어깨폭으로 나눈 값(정규화) + EMA로 부드럽게 한 값
        - MIN/MAX: 지금까지 관측된 D_SH_NORM(EMA) 의 최솟값/최댓값
            - MIN(음수): baseline보다 더 위로 잡힌 프레임이 있었음
            - MAX: 몸이 내려가며 어깨가 baseline 보다 꽤 아래로 간 프레임이 있었음
        - TH(D/U): 임계값 2개
            - UP → DOWN: 0.25
            - DOWN → UP: 0.10
        - BASE_SH_Y: baseline(UP 자세)로 잡힌 어깨 중앙 y 값
            - 화면 높이의 41.3% 지점에 어깨가 있었다는 의미
        - BASE_TORSO: baseline에서의 torso 길이 (어깨 중앙 ↔ 힙중앙 거리)
            - 푸쉬업 로직에서 정규화에 직접 사용하지 않음
        - BASE_SW: baseline 에서의 어깨폭 평균 값
            - 화면 너비의 약 8.9% 정도가 어깨폭 이라는 뜻
        - VIS(min4): 4개 관절(좌/우 어깨. 좌/우 힙)의 visibility 중 최솟값
    - 푸쉬업은 정면에서 보면 어깨가 위아래로 움직이는 것이 가장 명확하게 보임
        - **어깨가 기준 위치에서 얼마나 내려갔다가 다시 올라오는지를 보고 푸쉬업 횟수를 카운트**
    - 어깨가 내려간 정도를 내 어깨 폭으로 나눠서 카메라의 거리 등에 따른 변화 때문에 생기는 오차를 줄이며 자기 몸 기준으로 얼마나 내려갔는지를 계산

### 크런치

- 정면에서 크런치 1회를 상체를 들어 올리는 동작(DOWN → UP) 을 기준으로 카운트

**사용 관절**

- 2D visibility 체크용
    - 어깨(좌/우), 힙(좌/우), 무릎(좌/우) → 총 6개 visibility 최솟값
- 3D 각도 계산용
    - 2D 화면 좌표는 카메라 각도/거리/회전 영향이 커서 정면에서는 접힘이 정확히 반영이 안됨
    - 접힘을 반영해야 하는 크런치 동작에서는 3D로 몸의 깊이(z)까지 반영하는 각도 기반 판정이 더 안정적
    - 왼쪽 또는 오른쪽 관절들만 사용
        - LEFT_SHOULDER(11) / LEFT_HIP(23) / LEFT_KNEE(25)
        - 스쿼트 및 푸쉬업과 같이 신체의 상하 이동(translation)을 측정하는 운동에서는 좌·우 관절의 중앙값을 사용하여 신체 중심의 위치 변화를 계산함. 이는 좌우 비대칭이나 개별 랜드마크의 미세한 흔들림을 평균화하여 보다 안정적인 메트릭을 얻을 수 있음.
        - 반면, 크런치는 힙 관절의 회전량(rotation)을 각도로 측정하는 운동으로, 3차원 좌표 기반 각도 계산 시 좌우 관절을 혼합할 경우 프레임 간 기준축 변화로 인한 각도 점프 및 노이즈가 발생할 수 있음. 프레임 전반에 걸쳐 동일한 기준계를 유지하기 위해 좌·우 중 한쪽(LEFT) 관절만을 사용함.

**크런치 촬영각(90/60/45)**

- 크런치 동작은 접힘을 반영해야 하다 보니 정면(90도)에서는 정확한 카운팅이 어려울 수 있다고 판단
    - 정면보다는 상체 접힘이 보일 수 있는 정면보다 살짝 위의 각도에서가 더 안정적이지 않을까? 라는 생각을 함
    - blender 에서 카메라 위치를 60도, 45도로 설정하여 동일한 로직을 테스트
- 촬영각 평가 기준
    - `VIS(min6)`가 높고 안정적인가? (0.5 이상이 자주 유지되는가)
    - `SWING`이 충분히 큰가? (`MIN_SWING_DEG`보다 여유 있게 큰가)
    - `JUMP` 상황이 적은가? (갑자기 flex 튀어서 skip되는 프레임이 적은가)
    - HOLD로 상태 전환이 부드럽게 되는가? (UP/DOWN이 안정적으로 바뀌는가)
    - 실제 카운트가 사람이 느끼는 반복과 일치하는가?
- **생성 영상 실험**
    - 얼마나 위로 올라왔느냐 보다 **얼마나 접혔느냐 의 초점을 맞춤**
        - 스쿼트, 푸쉬업 → 관절의 위치 변화
        - 크런치 → 관절의 접힘 정도
    - 크런치 동작에서는 관절 각도(3D)를 계산
        - MediaPipe Pose 는 사람의 관절 위치(총 33개) 를 두 가지 형태로 제공
        
        | 구분 | 의미 | 특징 |
        | --- | --- | --- |
        | **2D 좌표** | 화면 기준 위치 | 0~1 사이 값 |
        | **3D 좌표** | 실제 공간 기준 위치 | 각도 계산 가능 |
        - 2D 좌표는 화면 기준 위치
            - x = 0 → 화면 맨 왼쪽
            - x = 1 → 화면 맨 오른쪽
            - y = 0 → 화면 맨 위
            - y = 1 → 화면 맨 아래
            - 예:`y = 0.53` → 이 관절은 화면 높이의 53% 지점에 있다.
            
            ```jsx
            if results.pose_landmarks:
            	lm = results.pose_landmarks.landmark
            ```
            
            - 사람이 위로 내려갔다/올라갔다 등의 위치 변화가 중요한 운동에 사용할 수 있음
        - 3D 좌표는 카메라 기준의 실제 공간 위치
            - x, y, z 값 제공
            
            ```jsx
            if results.pose_world_landmarks:
            	lm3d = results.pose_world_landmarks.landmark
            ```
            
            - 위치 변화보다 각도 변화가 중요한 운동 카운트에 사용
    - 어깨 - 힙 - 무릎 세 점으로 허리가 얼마나 접혔는지를 각도 계산
        - 접힘이 커지면 올라왔다, 다시 펴지면 내려갔다 판단
        - 고정 임계값을 사용하지 않고 (사람마다 올라오는 최대 각도나 체형 등이 다를 수 있기 때문) 이 사람이 이번에 얼마나 움직였는가? 를 계산
            - 가장 많이 올라왔을 때와 가장 많이 내려갔을 때의 차를 구하면 얼마나 움직였는지를 알 수 있음. 그 범위 안에서 일정 비율(35%) 이상 접히거나 펴졌을 때만 상태를 전환함.
                
                ### 절대 각도 기준의 문제
                
                - 만약 이렇게 하면? → “flex가 30° 넘으면 UP”
                    - A 사람은 최대 40°까지밖에 못 올라옴
                    - B 사람은 80°까지 올라옴
                    
                    같은 크런치인데:
                    
                    - A는 거의 끝까지 올라와도 카운트 안 됨
                    - B는 반만 올라와도 카운트 됨
                
                ### 상대 각도 기준
                
                ### 예시 1️⃣ (움직임이 작은 사람)
                
                - flex_min = 5°
                - flex_max = 25°
                - swing = 20°
                
                이 사람에게:
                
                - 25°는 **최대치**
                - 15°는 이미 꽤 올라온 상태
                
                그래서:
                
                ```
                up_th =5 +20 ×0.35 =12°
                ```
                
                👉 **12°만 넘어도 UP 인정**
                
                ### 예시 2️⃣ (움직임이 큰 사람)
                
                - flex_min = 10°
                - flex_max = 70°
                - swing = 60°
                
                이 사람에게:
                
                - 12°는 거의 안 움직인 것
                - 30°쯤 돼야 올라왔다 느낌
                
                그래서:
                
                ```
                up_th =10 +60 ×0.35 =31°
                ```
                
                👉 **같은 35% 기준이지만 절대값은 다름**
                
                - swing 을 알려면 이미 한 번의 동작이 필요하지 않나?
                    
                    코드 상: `WARMUP_FRAMES = 25` 
                    
                    - 처음 25 프레임은 학습 시간
                        - flex_min / flex_max 모으기
                        - swing 추정
                - 동작을 할 때마다 swing 을 계산해서 임계값이 바뀌니 운동을 하다 나중에 지치면 당연히 swing 값이 줄어들테고 그랬을 때 얼마 못올라와도 카운트가 되는 오류가 발생하지 않을까? 싶음 → 로직 수정이 필요할 것으로 예상
    
    ### 1) 45도
    
    ![image.png](attachment:da1a2803-e265-450d-9a64-8d21ce0da801:e677e99f-0fd9-4820-af45-35d087be2489.png)
    
    ![image.png](attachment:bdb6fbae-3268-4c9f-8cac-882baf0534de:daa0b09a-023b-43b6-8ed9-d0a08284e72d.png)
    
    - 화면 오버레이
        - state: 상태 (UP/DOWN)
        - count: 카운트 횟수
        - WARMUP: 워밍업 프레임 카운트 (25 프레임동안 값만 모으고 카운트는 잠깐 금지하는 시간)
        - HIP_FLEX(EMA): 현재 프레임의 힙 접힘 정도
            - flex가 클수록 상체가 많이 올라옴(많이 접힘) / flex가 작을수록 상체가 내려감(펴짐)
        - RAW_ANGLE: 원본 각도
            - 3D 점 3개(어깨-힙-무릎)로 만든 힙 관절 각도
        - FLEX_MIN/MAX: 최근 구간에서 관측된 flex의 최소/최대값
        - SWING: `flex_max - flex_min` → 최근 구간에서 접힘이 얼마나 움직였는지
            - MIN_SWING: 최소한 이 정도는 움직여야 진짜 크런치로 인정
        - TH UP/DN: 상태 전환 임계값 2개
        - HOLD up/dn: 연속 프레임 확인
            - HOLD_UP = 3: up 조건을 3프레임 연속 만족해야 UP 으로 인정
            - HOLD_DOWN = 3: down 조건을 3프레임 연속 만족해야 DOWN 으로 인정
            - 한 프레임 튐으로 상태가 바뀌는 걸 막음
        - ALL MIN/MAX FLEX: 영상 전체에서 관측된 flex의 최솟값/최댓값
        - VIS(min6): 6개 관절(양쪽 어깨/힙/무릎)의 visbility 중 최솟값
    
    ### 2) 60도
    
    ![image.png](attachment:9c738e0a-b6ce-4e9b-8b34-c865535d52e4:b97d135c-7c43-46f6-8d0b-2c2ce97428c1.png)
    
    ![image.png](attachment:8b625de2-0cf6-4a7a-8113-d5bef0f1f98f:8e3aa4ec-3f47-42f8-9dd1-6c963b3604c3.png)
    
    ### 3) 90도(정면)
    
    ![image.png](attachment:7b6b478e-1044-4cec-8704-ee186e44a0a3:e6603c3a-3edc-4b96-b4b3-77e086e2c627.png)
    
    ![image.png](attachment:7df7a936-38c5-4be9-8a13-866da57aac60:0b4be40b-ca6c-4af7-af01-e288f0fc31c1.png)
    
    ### 크런치 각도 설정
    
    - 크런치 카운팅에서 좋은 각도의 기준
    1. **UP / DOWN 차이가 숫자로 명확한가**
    2. **임계값을 잡기 쉬운가**
    3. **사용하는 관절을 잘 인식할 수 있는가**
    
    ---
    
    ### **가장 이상적인 각도: 45도~ 60도**
    
    - 45도 일 때
        - `ALL MIN/MAX FLEX: 71.3 / 127.9`
            - 범위는 꽤 넓음 → **동작 감지 잘 됨**
        - 위에서 내려다보는 느낌이 상대적으로 강해서 UP 근처에서는 각도 변화가 둔해질 수 있다는 점을 고려해야함
    - 60도 일 때
        - `ALL MIN/MAX FLEX: 20.2 / 121.8`
            - 범위 가장 큼 → **동작 감지 잘 됨**
        - 몸이 접히는 구조가 가장 입체적으로 잘 드러나는 각도
    - 90도 일 때
        - `ALL MIN/MAX FLEX: 60.2 / 141.5`
            - 예상보다 동작 감지가 어렵진 않은 것으로 보임
        - `VIS(min6)`이 **0.54까지 떨어짐**
            - 실사용에서는 얼굴/팔/어깨 겹침이나 조명, 옷, 체형에 따라 랜드마크 신뢰도 차이가 클 것으로 예상
    
    ---
    
    ### 권장 촬영 가이드
    
    > 카메라는 정면에서 약간 위쪽(대략 45~60도) 위치에 두고,
    상체·골반·무릎이 모두 보이도록 설치해주세요.
    가장 정확하게 운동 횟수를 측정할 수 있습니다.
    > 
    - 내부적으로는 **60도를 기준 값**으로 설계
- 결론
    - 45도/60도는 관절 겹침이 줄어 VIS가 안정적이었고 SWING이 크게 나오는 경향이 있었다.
    - 90도는 사람/동작에 따라 관절 겹침이 생기면 VIS가 흔들려 flex가 튀는 문제가 발생할 수 있다.

**로직**

- 세 점 A-B-C로 B가 꼭짓점인 각도를 계산 (0~180도)
    - A = shoulder / B = hip / C = knee
    - `flex = 180 - raw_angle` → 접힌 정도
        - `raw_angle` = 어깨-힙-무릎이 이루는 실제 관절 각도
            - `raw_angle = angle_abc_3d(Shoulder, Hip, Knee)`
        - 펴져 있으면 flex ≈ 0
        - 접히면 flex가 커짐
- 최근 관측한 최소/최대(flex_min, flex_max)로 현재 범위를 만들고 그 범위를 비율로 기준선을 잡음
    - `swing = flex_max - flex_min` → 이번 동작에서 얼마나 크게 움직였는지
    - swing이 `MIN_SWING_DEG = 12` 미만이면 아직 동작이 충분히 안 나온 것 → 카운트 금지
        - 이번 동작에서 최소한 이 정도 이상은 접혔다 펴져야 진짜 크런치로 인정한다의 의미
        - swing < 12° → 몸 거의 안 움직임 → 카운트 X
        - swing ≥ 12° → 의미 있는 크런치 → 카운트 가능
- 상태 전환 기준
    - DOWN → UP: `up_th = flex_min + swing * P_UP`
    - UP → DOWN: `down_th = flex_max - swing * P_DOWN`
        - 예) flex_min = 5° / flex_max = 35° → swing = 30°
            - up_th = 5 + 30 * 0.35 = 15.5°  → 가장 접히지 않은 상태에서 35% 정도 이상 접히면 올라왔다고 인정
            - down_th = 35 - 30 * 0.35 = 24.5°  → 최대한 접힌 상태에서 어느정도 다시 펴지면 내려왔다고 인정
    - 이번 사람의 움직임 범위 안에서 일정 비율만큼 올라오면 UP, 내려오면 DOWN

### 3.3 PC 환경에서 알고리즘 검증

### 3.3.1 스쿼트 실제 영상 TEST

[25.12.23 (실제 스쿼트 영상 test)](https://www.notion.so/25-12-23-test-2d2d4b8233fa80f69d5ee81c02813779?pvs=21) 

## 목표

- 이전에 생성한 시뮬레이션 영상을 기반으로 구현한 스쿼트 카운트 로직이 다양한 체형을 가진 실제 사람의 스쿼트 영상에서도 안정적으로 동작하는지를 검증한다.
- 실제 환경과 사람은 생성한 시뮬레이션 영상처럼 깔끔하거나 정확하지 않기 때문에 현재 로직에서의 한계점과 맨 몸 운동 카운팅 환경에서 고려해야 할 점 등을 알아보기 위함이다.
    1. 촬영 각도 및 위치
    2. 체형 차이에 대한 로직의 일반성 검증
    3. 임계 값 튜닝
    4. 실패 케이스
    5. 실제 사용 환경 고려
    
    → **향후 개선 방향 도출**
    

## 실험 환경

- 체형이 다른 사람 4명
- 연구소 내부 조명 아래
- S22 휴대폰 카메라
- 촬영 위치/각도 다르게 3 케이스
    - 각 케이스 별 스쿼트 5회 촬영
    1. 180cm 거리 / 바닥
    2. 50cm 거리 / 정면
    3. 50cm 거리 / 45도 위

## 평가 지표

- 카운트가 잘 되는가?
- 몸의 관절의 인식이 잘 되는가?
    - 팔, 몸 등 겹침 여부가 발생하는가?
    - 스켈레톤 인식이 안정적인가?
- UP/DOWN 판단을 위한 임계값 튜닝을 위해 사람 별 동작 크기가 어느 정도인가?
    - 사람 별 최소/최대 힙의 내려감/올라감이 어느 정도인가?

## 실험 결과

- 스쿼트 10회를 진행한다는 가정하에 카운트

https://drive.google.com/drive/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo?hl=ko 영상

https://docs.google.com/spreadsheets/d/1x5faVOD94T6PWvlAwtJkUqWIv4P6fMD3ICXDR3xTJTE/edit?gid=1379620090#gid=1379620090 구글 시트

### 1. 바닥 촬영

- 촬영 거리: 180cm / 촬영 각도: 휴대폰 바닥에 두고 정면 촬영

![image.png](attachment:4394e158-0288-4c2f-a9c9-9ca28f7c627c:image.png)

| 사람 | HIP 최소 | HIP 최대 | HIP range | baseline 힙 | baseline torso | 카운트 |
| --- | --- | --- | --- | --- | --- | --- |
| 소장님 | -0.014 | 0.944 | 0.958 | 0.368 | 0.139 | 10/10 |
| 세빈님 | 0.106 | 0.827 | 0.721 | 0.420 | 0.125 | 10/10 |
| 건우님 | 0.035 | 0.800 | 0.765 | 0.374 | 0.148 | 2/10 |
| 재진님 | -0.057 | 0.917 | 0.974 | 0.339 | 0.136 | 10/10 |
- 표 설명
    - **HIP 최소**
        - 가장 서 있거나, 가장 위로 올라왔을 때 힙 위치
        - 0에 가까울수록 baseline이 잘 잡혔다는 뜻
        - 음수 → baseline보다 더 위로 올라간 프레임이 있었다는 의미
    - **HIP 최대**
        - 가장 깊게 앉았을 때 힙이 내려간 정도
        - 클수록 → 깊은 스쿼트
    - **HIP range = 최대 − 최소**
        - **이 사람이 한 동작에서 얼마나 크게 움직였는지**
        - 스쿼트 동작 크기
    - **baseline 힙**
        - 처음에 이 사람이 서있다고 판단하고 측정된 힙의 기준 위치
        - 사람마다 키, 서 있는 위치, 카메라와의 거리가 모두 다름
        - baseline 힙 = 0.386
            - 화면 높이를 1 이라고 봤을 때 힙이 위에서 약 36.8% 지점에 있다는 뜻
            - 작을수록 힙이 화면 위쪽, 클수록 힙이 화면 아래쪽
        - 얼마나 내려갔는지를 이 값을 기준으로 계산: 현재 힙 위치 - baseline 힙 위치
    - **baseline torso**
        - 이 사람의 몸 크기를 나타내는 기준 길이 (어깨 중심 ~ 힙 중심 사이의 거리)
            - 화면 좌표(0~1)로 측정된 값
        - baseline torso = 0.139
            - 화면 높이의 약 13.9%가 이 사람의 몸통 길이
            - torso 값이 클수록 화면에서 몸이 크게 잡혔다는 뜻이고 torso 값이 작을수록 화면에서 몸이 작게 잡혔다는 뜻
        - 같은 동작이라도 키 큰 사람이 카메라 가까이 서면 움직임이 크게 보이고 키 작은 사람이 멀리 서면 움직임이 작게 보이기 때문에 키, 체형, 카메라 거리 등의 환경 차이를 줄이고 **‘내 몸 기준으로 얼마나 내려갔는지’**  보기 위함
1. 카운트가 잘 되는가?
    - 총 40번의 카운트 (4명 * 10회) 중 **32번**이 제대로 카운트 됨 → **약 80% 정확도**
2. 몸의 관절의 인식이 잘 되는가?
    - 정면이 아닌 바닥 촬영이나 위 각도에서 촬영을 하게 되면 어쩔 수 없는 왜곡이 생기게 되는데 바닥 촬영의 경우, 심하진 않지만 종종 무릎의 랜드마크가 끊기는 경우가 있었음
        - 현재 스쿼트 카운링 로직에서는 무릎의 위치 변화 등 무릎 인식과 상관 없기 때문에 큰 문제가 되지는 않지만 스쿼트 카운팅 로직을 무릎과 관련하여 변경한다면 고려 필요할 것으로 보임
    - 각 스켈레톤이 겹쳐 관절 위치 인식이 어려운 경우는 없었음
3. 임계값 튜닝의 필요성
    - **실제 데이터 범위: 약 0.15~0.8**
    - 코드 상 임계 값
        - 0.18보다 크면 DOWN → 범위 초입 (매우 관대)
        - 0.08보다 작으면 UP → 매우 빡셈
    - 임계값 튜닝이 필요해보임. 특히 UP 조건

![image.png](attachment:e129dabe-d68b-4495-bdaa-4a69ba58b255:image.png)

![image.png](attachment:f33ade87-2500-4aa8-b925-49e72aa2514b:image.png)

![image.png](attachment:bf54b35d-1b71-4718-b18d-2b388cd741f0:image.png)

![image.png](attachment:49b91e57-e3d7-4e36-b6b0-d9eebb069df2:image.png)

![image.png](attachment:61298f26-7100-4b6d-bb30-b3b33bf358c5:image.png)

![image.png](attachment:02b8d331-c553-4627-8f25-20797a6b48c4:image.png)

![image.png](attachment:0eb91ed7-4a96-4634-8fbe-d89ce8c3cd0a:image.png)

![image.png](attachment:665c865d-a32a-462c-8090-5a73664c2dd1:image.png)

### 2. 정면 중간 위치 촬영

- 촬영 거리: 50cm / 촬영 각도: 중간 위치 정면 촬영

![image.png](attachment:6fa4b52d-2ae0-48b7-af67-ee5aff2b9df1:image.png)

| 사람 | HIP 최소 | HIP 최대 | HIP range | baseline 힙 위치 | baseline torso | 카운트 |
| --- | --- | --- | --- | --- | --- | --- |
| 소장님 | -0.021 | 0.913 | **0.934** | 0.436 | 0.207 | 10 / 10 |
| 세빈님 | 0.013 | 1.384 | **1.371** | 0.425 | 0.201 | 3 / 10 |
| 건우님 | 0.023 | 0.698 | **0.675** | 0.462 | 0.230 | 0 / 10 |
| 재진님 | 0.039 | 1.010 | **0.971** | 0.445 | 0.221 | 2 / 10 |
- 표 설명
    - **HIP 최소**
        - 가장 서 있거나, 가장 위로 올라왔을 때 힙 위치
        - 0에 가까울수록 baseline이 잘 잡혔다는 뜻
        - 음수 → baseline보다 더 위로 올라간 프레임이 있었다는 의미
    - **HIP 최대**
        - 가장 깊게 앉았을 때 힙이 내려간 정도
        - 클수록 → 깊은 스쿼트
    - **HIP range = 최대 − 최소**
        - **이 사람이 한 동작에서 얼마나 크게 움직였는지**
        - 스쿼트 동작 크기
    - **baseline 힙**
        - 처음에 이 사람이 서있다고 판단하고 측정된 힙의 기준 위치
        - 사람마다 키, 서 있는 위치, 카메라와의 거리가 모두 다름
        - baseline 힙 = 0.386
            - 화면 높이를 1 이라고 봤을 때 힙이 위에서 약 36.8% 지점에 있다는 뜻
            - 작을수록 힙이 화면 위쪽, 클수록 힙이 화면 아래쪽
        - 얼마나 내려갔는지를 이 값을 기준으로 계산: 현재 힙 위치 - baseline 힙 위치
    - **baseline torso**
        - 이 사람의 몸 크기를 나타내는 기준 길이 (어깨 중심 ~ 힙 중심 사이의 거리)
            - 화면 좌표(0~1)로 측정된 값
        - baseline torso = 0.139
            - 화면 높이의 약 13.9%가 이 사람의 몸통 길이
            - torso 값이 클수록 화면에서 몸이 크게 잡혔다는 뜻이고 torso 값이 작을수록 화면에서 몸이 작게 잡혔다는 뜻
        - 같은 동작이라도 키 큰 사람이 카메라 가까이 서면 움직임이 크게 보이고 키 작은 사람이 멀리 서면 움직임이 작게 보이기 때문에 키, 체형, 카메라 거리 등의 환경 차이를 줄이고 **‘내 몸 기준으로 얼마나 내려갔는지’**  보기 위함
1. 카운트가 잘 되는가?
    - 총 40번의 카운트 (4명 * 10회) 중 **15번**이 제대로 카운트 됨 → **약 37.5% 정확도**
2. 몸의 관절의 인식이 잘 되는가?
    - 각 관절의 랜드마크가 튀는 경우는 거의 없지만 DOWN 과정에서 손이나 팔 등이 힙 좌표를 가리는 경우가 종종 발생
        - 그렇게 되면 힙의 위치나 변화를 판단하여 계산할 수 없기 때문에 카운팅에 직접적인 영향을 미침
3. 임계값 튜닝의 필요성
    - **실제 데이터 범위: 약 약 0.02 ~ 1.38**
    - 코드 상 임계 값
        - 0.18보다 크면 DOWN → 범위 초입 (매우 관대)
        - 0.08보다 작으면 UP → 범위 매우 빡셈

![image.png](attachment:18230a55-6b82-4cd8-895a-530d59fefb6c:image.png)

![image.png](attachment:8dc88f69-d449-490a-8a7c-c7022e7e5ec4:image.png)

![image.png](attachment:bbecb344-6708-486d-9b69-6d3faebf5751:image.png)

![image.png](attachment:94469e88-ce40-4a3d-8291-b51c1bef9294:image.png)

![image.png](attachment:825c84c5-e091-44b4-acc5-736c0b04412c:image.png)

![image.png](attachment:d413ce0f-c2b7-46df-a030-b7ba585baa85:image.png)

![image.png](attachment:f1aeb521-ac8b-4a8b-9d25-8c7c6a9d63aa:image.png)

![image.png](attachment:e8a59c5f-65b9-49fa-99ac-8ead5ae1fb3e:image.png)

### 3. 정면 45도 위 촬영

- 촬영 거리: 50cm / 촬영 각도: 정면보다 위 쪽에서 45도 각도로 촬영

![image.png](attachment:a408d7ff-4b8d-4b45-98b6-d777dc990299:image.png)

| 사람 | HIP 최소 | HIP 최대 | HIP range | baseline 힙 | baseline torso | 카운트 |
| --- | --- | --- | --- | --- | --- | --- |
| 소장님 | 0.009 | 1.063 | 1.054 | 0.471 | 0.250 | **2/10** |
| 세빈님 | -0.151 | 0.922 | 1.073 | 0.525 | 0.211 | **10/10** |
| 건우님 | 0.003 | 0.648 | 0.645 | 0.494 | 0.255 | **5/10** |
| 재진님 | 0.011 | 0.640 | 0.629 | 0.530 | 0.208 | **10/10** |
- 표 설명
    - **HIP 최소**
        - 가장 서 있거나, 가장 위로 올라왔을 때 힙 위치
        - 0에 가까울수록 baseline이 잘 잡혔다는 뜻
        - 음수 → baseline보다 더 위로 올라간 프레임이 있었다는 의미
    - **HIP 최대**
        - 가장 깊게 앉았을 때 힙이 내려간 정도
        - 클수록 → 깊은 스쿼트
    - **HIP range = 최대 − 최소**
        - **이 사람이 한 동작에서 얼마나 크게 움직였는지**
        - 스쿼트 동작 크기
    - **baseline 힙**
        - 처음에 이 사람이 서있다고 판단하고 측정된 힙의 기준 위치
        - 사람마다 키, 서 있는 위치, 카메라와의 거리가 모두 다름
        - baseline 힙 = 0.386
            - 화면 높이를 1 이라고 봤을 때 힙이 위에서 약 36.8% 지점에 있다는 뜻
            - 작을수록 힙이 화면 위쪽, 클수록 힙이 화면 아래쪽
        - 얼마나 내려갔는지를 이 값을 기준으로 계산: 현재 힙 위치 - baseline 힙 위치
    - **baseline torso**
        - 이 사람의 몸 크기를 나타내는 기준 길이 (어깨 중심 ~ 힙 중심 사이의 거리)
            - 화면 좌표(0~1)로 측정된 값
        - baseline torso = 0.139
            - 화면 높이의 약 13.9%가 이 사람의 몸통 길이
            - torso 값이 클수록 화면에서 몸이 크게 잡혔다는 뜻이고 torso 값이 작을수록 화면에서 몸이 작게 잡혔다는 뜻
        - 같은 동작이라도 키 큰 사람이 카메라 가까이 서면 움직임이 크게 보이고 키 작은 사람이 멀리 서면 움직임이 작게 보이기 때문에 키, 체형, 카메라 거리 등의 환경 차이를 줄이고 **‘내 몸 기준으로 얼마나 내려갔는지’**  보기 위함
1. 카운트가 잘 되는가?
    - 총 40번의 카운트 (4명 * 10회) 중 **27번**이 제대로 카운트 됨 → **약 67.5% 정확도**
2. 몸의 관절의 인식이 잘 되는가?
    - 각 관절의 랜드마크의 튐이 가장 심하고 앉는 과정에서 관절의 랜드마크가 겹치는 부분이 빈번하게 발생
    - 위 쪽에서 촬영하여 몸 통의 위치 변화율은 상대적으로 커보이고 힙의 위치 변화율은 작아보임. 즉 힙의 위치 변화를 잘 반영하지 못하는 거 같음
3. 임계값 튜닝의 필요성
    - **실제 데이터 범위: 약 -0.15 ~ 1.05**
    - 코드 상 임계 값
        - 0.18보다 크면 DOWN → 범위 초입 (매우 관대)
        - 0.08보다 작으면 UP → 범위 빡셈

![image.png](attachment:ed272567-1779-405c-90c4-b9613c406b54:image.png)

![image.png](attachment:2ffaec0f-c25c-4a08-890b-6c8e7308f823:image.png)

![image.png](attachment:c562b969-faf8-49ac-9c00-cc10450ae71a:image.png)

![image.png](attachment:77145b23-a08a-470f-a2ed-9e76f9cc012d:image.png)

![image.png](attachment:a13b9f47-8a81-4a6b-af07-91cd0fce68c5:image.png)

![image.png](attachment:5e767647-73b8-4d67-b3f8-aaf13690535f:image.png)

![image.png](attachment:37098de3-a383-41fd-ab2c-5ad18cf178a8:image.png)

![image.png](attachment:1df6d2e7-3239-4b69-89a9-e3d105889605:image.png)

## 결론

1. 촬영 각도 및 위치
    - 임계값 튜닝 후 다시 확인을 해봐야겠지만 현재까지는 바닥 촬영이 가장 인식이 잘 되어보임
        - 손, 팔 등이 힙을 가리는 경우가 없고 힙의 위치 변화가 잘 보임
        - 하체 랜드마크(무릎, 발)가 끊기는 케이스가 종종 발생하지만 카운팅에 직접적인 영향을 미치진 않음
2. 체형 차이에 대한 로직의 일반성 검증
    - 힙의 변화량 / 몸 통 길이 로 정규화를 하고 있어 체형 별, 환경 별 영향을 거의 받지 않아 스케일 차이에는 비교적 강함
3. 임계값 튜닝
    - 절대값(0.18/0.08) 고정 말고 해당 영상의 range(최소~최대)를 이용해 임계값을 비율로 잡아 몇 % 이상 일어나면 UP, 앉으면 DOWN 로직으로 설정할 수 있음
        - 이 경우에는 사람마다/각도마다의 영향은 덜 받겠지만
            - range를 언제까지 측정할 지 (프레임 기준, 몇 초 기준 등)
            - UP, DOWN 을 허용하는 %를 몇 으로 설정할 지 등 고려 필요
    - 촬영 각도나 위치가 고정되면 그거에 맞춰 여유있는 절대값으로 고정
4. 실패 케이스
    - 포즈 인식 자체가 불안정
    - 임계값이 안맞음
    - 손, 팔 등이 어깨나 힙을 가려 랜드마크 추출이 안됨
5. 실제 사용 환경 고려
    - 카운팅 시작 전 안내,
    - 사용자에게 손, 팔 위치를 안내하거나
    - 촬영 각도, 위치를 안내하는 등의 **권장 촬영 가이드** 를 제공하여야 함

### 3.3.2 정면 스쿼트 로직: 무릎 각도 대신 힙 하강량

https://github.com/pnkmem432/workout-counter-pc/tree/v0.4/v0.4

https://drive.google.com/drive/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo?hl=ko

‣ 

- 정면에서 스쿼트는 무릎 각도보다 사람 전체가 아래로 내려갔다가 올라오는 변화가 더 명확함

**메트릭**

- 힙 중앙 y 좌표: `mid_hip_y = (L_HIP.y + R_HIP.y)/2`
- baseline(서 있을 때 기준): 시작 초반 프레임 평균으로 `hip0_y` 저장
- 내려간 정도: `drop = mid_hip_y - hip0_y`

**정규화**

같은 drop 이라도 키 큰 사람이 가까이 서면 크게 보이고, 키 작은 사람이 멀리 서면 작게 보이므로 drop을 몸통 길이로 나눠 내 몸 기준 비율로 변환

- `torso_len = distance(mid_shoulder, mid_hip)`
- `depth_pct = drop/torso_len`

이 값으로 DOWN/UP 전이를 정의하여 카운트 

### 3.3.3 정면 푸쉬업 로직: 팔꿈치 각도 대신 어깨 하강량

https://github.com/pnkmem432/workout-counter-pc/tree/v0.4/v0.4

https://drive.google.com/drive/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo?hl=ko

‣ 

- 정면 푸쉬업에서는 팔꿈치 각도가 잘 보이지 않을 수 있어 어깨의 상하 움직임이 더 안정적일 것이라 생각함

**메트릭**

- 어깨 중앙 y: `mid_sh_y`
- baseline(UP 기준): UP 자세를 자동 탐지해 `sh0_y` 확보
- 내려간 정도: `d = mid_sh_y - sh0_y`

**정규화**

- 푸쉬업은 몸 전체가 아래로 움직이면서도 화면 크기가 변할 수 있기 때문에 내려간 정도를 어깨폭으로 나눠 정규화
- `d_norm = (mid_sh_y - sh0_y) / shoulder_width`

이 값을 기반으로 DOWN/UP 전이를 정의하여 카운트 

### 3.3.4 크런치 로직: 위치 변화가 아닌 접힘

https://github.com/pnkmem432/workout-counter-pc/tree/v0.4/v0.4

https://drive.google.com/drive/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo?hl=ko

‣ 

- 크런치는 스쿼트/푸쉬업처럼 위치가 내려갔다 올라오는 운동이 아니라 몸통이 얼마나 접혔는지가 핵심
- 정면에서 2D 좌표만 사용하면 카메라 투영의 영향이 커서 각도가 불안정할 수 있어 `pose_world_landmarks(3D)`를 활용

**메트릭**

- 어깨-힙-무릎 3점 각도 계산 → `raw_angle`
- 접힘 정도: `flex = 180 - raw_angle`

**상대 범위 기반**

사람마다 접힘 정도의 범위가 다르기 때문에 절대 기준이 아닌 

- `flex_min, flex_max`
- `swing = flex_max - flex_min`

을 만들고, 그 범위에서 일정 비율 이상 접히면 UP으로 인정하는 방식을 사용

### 3.4 모바일 환경에서 구현

- PC 에서 로직을 검증한 후, 실제 사용자가 사용할 수 있는 형태를 목표로 모바일 앱으로 이식

### 3.4.1 모바일에서의 구조 설계 (Flutter ↔ Kotlin)

‣ 

모바일에서는 MediaPipe PoseLandmarker(Task)가 Android 네이티브(Kotlin)에서 안정적으로 동작하므로 역할을 다음과 같이 분리

- **Flutter(Dart)**
    - 카메라 프리뷰 및 UI 표시
    - 운동 선택 / 카운트 화면 / 안내 문구
    - 카운팅 로직 수행
    - 스켈레톤 오버레이 렌더링
- **Android Kotlin**
    - MediaPipe PoseLandmarker 실행
    - 33개 관절 좌표 계산
    - 결과를 Flutter로 스트리밍 전송

통신은

- MethodChannel(프레임 전달)
- EventChannel(랜드마크 스트리밍)

### 모바일 앱 UI 설계

![login.png](attachment:80df74e2-26c5-4875-a984-0e4e0e69e1ae:login.png)

![homescreen.png](attachment:3c89b9b4-17a7-40b3-a199-56ef9fcf64a4:homescreen.png)

![select.png](attachment:6c0fd69e-1362-41f9-8290-42611b928d82:select.png)

![squat.png](attachment:abfaef77-cd49-4741-9656-f42b36e5c411:squat.png)

![log (1).png](attachment:51abf5c4-877c-4888-aa13-4c8596365035:log_(1).png)

![log particular.png](attachment:dd38ae00-08ce-4d77-892e-89e048bdea0d:log_particular.png)

### 3.4.2 모바일 운동별 구현 확장

- 모바일 환경에서는 PC 환경과 달리 카메라 노출 자동 보정, 조명 변화 등으로 인해 MediaPipe 포즈 인식이 더 불안정해질 수 있다. 따라서 모바일에서 운동 카운팅을 구현할 때, 단순히 카운트가 되는지를 넘어서 실제 사용자가 반복 사용해도 안정적으로 동작하도록 다음 원칙을 적용한다.
    1. **READY → CALI → COUNT 의 3단계 구조**
        - 잘못된 자세에서 기준값이 저장되면 이후 카운트가 무너짐 → READY
        - 사람마다 체형/카메라 거리/자세 범위가 다름 → CLAI
        - 카운팅은 상태머신으로 정의 → COUNT 단계에서만 카운트 수행
    2. **포즈 품질 게이트(poseOK/visibility)와 안정화(EMA/hold/miss)를 항상 사용**
        - 오탐을 줄이기 위해 잘 보이는 프레임에서만 계산
        - 프레임의 순간 튐을 EMA로 완화하고, 상태 전이는 연속 프레임 유지 조건으로 확정
    3. **운동별 핵심 관절/핵심 메트릭을 다르게 선택**
        - 스쿼트/푸쉬업은 정면에서 상하 이동이 더 명확
        - 크런치는 누움 기준 유지 + 굴곡(접힘)이 더 중요
        - 버피는 단일 조건이 아니라 순서가 있는 단계 운동이라 상태 머신을 세분화

### 스쿼트

https://github.com/pnkmem432/workout-counter-mobile/tree/squat/lib

https://drive.google.com/drive/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo?hl=ko → `모바일_스쿼트.mp4`  참고

‣ 

### 목표

- 스마트폰 전면 카메라에서 사용자가 스쿼트를 수행하면 엉덩이가 내려갔다가 다시 올라오는 1회 동작을 안정적으로 감지하여 자동 카운트

### READY 설계

- 스쿼트는 서 있는 자세를 기준으로 함
1. POSE_OK: 스쿼트에 중요한 관절(어깨/무릎/힙)의 visibility가 충분히 높음
    - 핵심 6개 관절(좌/우 어깨, 좌/우 힙, 좌/우 무릎)의 visibility 중 최솟값 ≥ `visTh(0.5)`
2. STAND_OK: 진짜 서 있는 자세인지 판별
    - `hipY < kneeY - margin` 형태로 힙이 무릎보다 충분히 위 조건 적용
3. STABLE_OK: 가만히 서 있는 상태가 유지되는지 확인
    - 최근 N프레임 동안 hip 높이 변화량과 기준 길이 변화량이 너무 크면 READY 실패

위 조건을 연속 N 프레임 유지하면 READY 통과

### CALI 설계

- READY 가 통과되면 3초동안 서 있는 데이터를 모아 평균을 내어 다음 기준 값을 저장
- `hip0Y`: 서 있을 때 힙 중앙 y 평균(서 있는 기준 위치)
- `refLen0`: 어깨~무릎 길이(내 몸 스케일 기준 길이)

### COUNT 로직

- 스쿼트 카운팅 핵심은 힙이 기준 대비 얼마나 내려갔는가
    - `drop = midHipY - hip0Y`
    - `depthPct = drop / refLen0` (정규화)
    - `emaPct = EMA(depthPct)` (노이즈 완화)
- 상태 머신:
    - **UP → DOWN:** `emaPct ≥ downThPct(0.25)`
    - **DOWN → UP (+1):** `emaPct ≤ upThPct(0.12)`이면 count 증가

### 푸쉬업

https://github.com/pnkmem432/workout-counter-mobile/tree/pushup/lib

https://drive.google.com/drive/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo?hl=ko → `모바일_푸쉬업.mp4` 참고

[26.01.05 (모바일_푸쉬업 구현)](https://www.notion.so/26-01-05-_-2dfd4b8233fa80b487addf3ff479e053?pvs=21) 

### 목표

- 푸쉬업은 엎드린 UP 자세가 기준 자세이므로, 서 있는 상태에서 READY/CALI가 시작되면 로직이 무너짐으로 푸쉬업 자세 판별을 통한 정확한 카운팅을 목표로 함

### READY 설계

1. poseOK: 어깨/팔꿈치 관절이 잘 보이는지
    - 좌/우 어깨 + 좌/우 팔꿈치 visibility 최솟값 ≥ `0.6`
2. postureOK: 지금 자세가 푸쉬업 자세인지
    - (1) 몸이 세워졌는지/눕혀졌는지(bodyRatio)
        - 어깨~힙의 세로 폭(spreadY) / 가로 폭(spreadX)
        - 눕혀져 있으면 bodyRatio가 작아짐 → `bodyRatio < 0.8`
    - (2) 손 지지 여부(handSupport)
        - 손이 어깨보다 아래에 위치하는지 등 조건으로 바닥 지지 확인
3. stableOK: 최근 N 프레임동안 어깨 높이, 너비 변화가 작음

### CALI 설계

- READY가 연속으로 유지되면 CALI를 수행하여 다음의 값 저장
    - sh0Y: UP 자세에서 어깨가 가장 위에 있는 y 값
    - sw0: 평균 어깨 너비

### COUNT 로직

- `d = (midShoulderY - sh0Y) / sw0`
- `emaD = EMA(d)`

FSM:

- **UP → DOWN:** `emaD ≥ 0.30`
- **DOWN → UP (+1):** `emaD ≤ 0.25`

### 크런치

https://github.com/pnkmem432/workout-counter-mobile/tree/crunch/lib

https://drive.google.com/drive/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo?hl=ko — `모바일_크런치.mp4` 영상 참고

‣ 

### 목표

- 크런치는 누워서 상체를 들었다가 다시 누움을 1회로 카운팅 하는 로직 필요

### READY 설계

1. **프레임 유효성(ok):** Kotlin 측에서 유효 프레임인지 판단
2. **관절 품질(vis):** 어깨/골반/무릎(좌우 6개) min_vis ≥ 0.60
3. **누워있는 자세 판별**
    - (1) 어깨–골반 높이 차이: `|shoulderY - hipY| ≤ 0.22`
    - (2) 상체 기울기 각도: 누운 상태에 해당하는 범위 허용

위 조건이 연속 10프레임 유지되면 READY 통과

### 3) CALI

- `downFlex0`: 이 사람이 가만히 누워있을 때의 평균 flex(기준 DOWN)

### 4) COUNT 로직

1. **DOWN → UP**
    - `최소 움직임 ≥ 12°` + `UP 유지 ≥ 3프레임`
    - 여기서는 올라왔다 가능성만 기록(pending)
2. **UP → DOWN (확정)**
    - 다시 누운 상태 조건 통과(어깨–골반 높이 차이)
    - `|flex - downFlex0| ≤ 18°`
        
        → 일어서는 동작을 크런치로 오인하지 않게 막음
        
3. 조건이 3프레임 유지되면 `count += 1`

### 촬영 위치 테스트

- 데모 이후 촬영 위치 테스트 필요성을 느끼고 실험 설계를 진행 — ‣
- 촬영 위치 실험 과정 및 결과 정리 — ‣

### 목적

- 모바일에서 운동 카운팅을 구현하던 중, 같은 코드인데도 카메라 위치(높이)와 조명(light/dark)에 따라
    - 스켈레톤이 끊기거나
    - READY/CALI가 늦어지거나 실패하고
    - 카운트가 정상적으로 되지 않는 현상 반복적으로 발생
- 즉, 알고리즘 문제만이 아니라 촬영 환경이 성능에 큰 영향을 줄 수 있겠다라는 생각을 하게 됨
- 본 실험의 목적은 다음과 같다.
    1. 카메라 높이/조명이 MediaPipe Pose 품질에 어떤 영향을 주는 지 확인
    2. 운동 카운팅 정확도가 환경에 따라 얼마나 달라지는지 비교
    3. 실사용을 위한 권장 촬영 가이드(높이/조명)을 도출

### 실험 설계

**(1) 변수 1: 카메라 높이(위치)**

모든 경우 전신 스켈레톤이 프레임에 들어오는 것을 전제로, 아래 3가지 높이를 비교

- **bottom(바닥)**: 휴대폰을 바닥에 둠
- **65cm**: 무릎~골반 높이(삼각대)
- **90cm**: 배~가슴 높이(삼각대)
- 상단(내려다보는 각도)은 실제 혼자 운동 시 현실성이 낮아 제외 방향으로 고려

**(2) 변수 2: 조명**

- **Light**: 연구소 전등 ON
- **Dark**: 연구소 전등 OFF

**(3) 운동 종류**

- 스쿼트 5회
- 푸쉬업 5회
- 크런치 5회

**(4) 전체 조건 수**

- 운동 3종 × 높이 3종 × 조명 2종 = **총 18조건**

### 실험 결과 요약

**공통 결론**

1. Light 환경이 Dark 환경보다 항상 안정적
2. bottom(바닥) 환경은 3가지 운동 모두에서 부적합
3. 운동마다 최적 높이가 조금은 다를 수 있음

**권장 촬영 가이드**

**케이스 1: 세 운동을 한 위치로 고정해야 한다면—** 65cm 권장

**케이스 2: 운동 별로 최적 위치를 다르게 둘 수 있다면—**

스쿼트 90cm / 푸쉬업 65cm / 크런치 65cm 

| 운동 | 카메라 위치 | Light 환경 | Dark 환경 | 종합 평가 | 비고 |
| --- | --- | --- | --- | --- | --- |
| 스쿼트 | 90cm | ⭐⭐⭐⭐☆ | ⭐⭐⭐☆☆ | ✅ **최적** | CALIB 빠름 |
| 스쿼트 | 65cm | ⭐⭐⭐☆☆ | ⭐⭐⭐☆☆ | ⚠️ 가능 | vis 감소 |
| 스쿼트 | bottom | ⭐⭐☆☆☆ | ⭐☆☆☆☆ | ❌ 부적합 | CALIB 실패 잦음 |
| 푸쉬업 | 90cm | ⭐⭐☆☆☆ | ⭐☆☆☆☆ | ❌ 비권장 | 중간 끊김 |
| 푸쉬업 | 65cm | ⭐⭐⭐⭐☆ | ⭐⭐⭐☆☆ | ✅ **최적** | 카운트 안정 |
| 푸쉬업 | bottom | ⭐☆☆☆☆ | ⭐☆☆☆☆ | ❌ 부적합 | 서서 인식 불가 |
| 크런치 | 90cm | ⭐☆☆☆☆ | ⭐☆☆☆☆ | ❌ 비권장 | READY 오동작 |
| 크런치 | 65cm | ⭐⭐⭐☆☆ | ⭐⭐☆☆☆ | ⚠️ 가능 | 포즈 유지 필요 |
| 크런치 | bottom | ⭐☆☆☆☆ | ⭐☆☆☆☆ | ❌ 부적합 | 카운트 실패 |

### 버피테스트

https://github.com/pnkmem432/workout-counter-mobile/tree/burpee/lib

https://drive.google.com/drive/folders/1tFQeYSDucpI86xzi6u9JM7Fk9BBMv8X0?hl=ko — `모바일_버피 2차(1).mp4` ,`모바일_버피 2차(2).mp4`  참고

[26.01.08 (모바일_버피 v1 구현)](https://www.notion.so/26-01-08-_-v1-2e2d4b8233fa80729ad8d1f97448c028?pvs=21) 

[26.01.09 (모바일_버피 v2 구현)](https://www.notion.so/26-01-09-_-v2-2e3d4b8233fa80ef91c5caf7ec0037f3?pvs=21) 

- 버피테스트는 다른 3가지 운동과 다르게 여러 동작을 순서대로 해야 1회가 되는 전신운동이므로 단일 기준값이 아닌 상태 전이 를 기반으로 설계

### 버피 v1 구현

- STAND → DOWN → PLANK → BACK → STAND(+1) 형태로 구현
- 기본 캘리: `refLen0(어깨~무릎)`, `hipStandY`, `ankleStandY` 저장
- 핵심 메트릭:
    - `hipDrop = (midHipY - hipStandY) / refLen0`
    - `angleHip = angle(midSh, midHip, midKnee)`

**v1에서 확인된 문제**

- 스쿼트만 해도 버피로 카운트되는 오탐이 발생
    
    → 즉, 버피의 핵심 단계(PLANK, 손 지지)를 더 강하게 요구해야 했음
    

### 버피 v2 구현

- **STAND → DOWN → PLANK → SQUAT → STAND** 순서를 맞춰야만 1회를 카운트 하도록 해야 함

**핵심 개선점**

1. **단계별 visTh를 다르게 적용**
    - 시작 전(STAND/READY): 더 엄격(0.50)
    - 진행 중(STAND): 완화(0.45)
    - DOWN/PLANK/SQUAT: 더 완화(0.40)
        
        → 단계 특성상 불안정한 구간을 현실적으로 반영
        
2. **DOWN → PLANK 진입 조건 강화(손 지지 + 수평 + 각도)**
    - 손목/팔꿈치 visibility 조건
    - 손목이 어깨보다 충분히 아래
    - 어깨~손목 거리(팔이 펴진 느낌)
    - 몸통 수평(torsoLevel)
    - 힙 범위(너무 작은/큰 값 제거)
    - 조건을 연속 프레임으로 만족해야 PLANK 진입
3. **PLANK 유지 조건(히스테리시스) 추가**
    - 순간 튐으로 바로 단계가 깨지지 않도록 최근 프레임 누적 기반으로 `handSupportKeep` 판단
4. **PLANK → SQUAT 전이 2트랙**
    - 다리가 잘 보이는 경우: knee/ankle이 hip 근처로 들어오는 조건
    - 다리가 잘 안 보이는 경우(정면/가림): `torsoLevel`/`hipDrop`/`handSupport`조건 기반 대체 전이
5. **SQUAT → STAND에서만 최종 카운트 확정**
    - 다시 서 있는 값(`standHipDropMax`)으로 돌아왔을 때만 +1

**결과**

- v1에서 발생하던 오탐(스쿼트만 해도 버피 카운트)을 대부분 차단
- 체감 기준 **약 10회 중 8회 수준**으로 카운트 안정성 개선

### 버피테스트 촬영 위치 테스트

‣ 

https://drive.google.com/drive/folders/1oq7H9Lf2DxzlWio7GbVDsjr4aT-QH-Dn?hl=ko

 https://github.com/pnkmem432/workout-counter-mobile/blob/extras/lib/pose_live_screen.dart https://github.com/pnkmem432/workout-counter-mobile/blob/extras/lib/counters/burpee_counter.dart

### 목적

1. 카메라 높이(촬영 위치)와 조명(light/dark)이 버피 카운팅에 미치는 영향 확인
2. 버피에서 중요한 구간(특히 PLANK)의 Pose 품질 저하/끊김이 얼마나 발생하는지 분석
3. 최종적으로 실사용 권장 촬영 환경(높이/조명)을 결정하고 가이드로 제시

### 실험 설계

**(1) 촬영 위치(카메라 높이)**

- **bottom(바닥)**
- **65cm(무릎~골반 높이, 삼각대)**
- **90cm(배~가슴 높이, 삼각대)**
- 상단(내려다보는 각도)은 혼자 운동 시 현실성이 낮아 이번 실험에서는 제외 방향으로 고려

**(2) 조명 환경**

- **Light**: 연구소 전등 ON
- **Dark**: 연구소 전등 OFF

**(3) 운동 수행**

- 각 조건에서 버피 **5회 수행**

**(4) 비교 지표**

1. **Pose 품질**(포즈가 끊기는지/흔들리는지)
2. **카운트 정확도**(5회 중 몇 회 성공)
3. **False Ready**(준비 상태에서 잘못 시작되는지)

### 실험 결과

**공통 관찰**

1. Light 환경이 Dark 환경보다 항상 안정적
2. bottom(바닥) 환경은 버피에서 실사용 어려움
    - 초기 포즈 인식 실패 + PLANK 단게에서 포즈 튐으로 카운트 불가

**결론**

추천 촬영 환경: 카메라 높이 65cm

- 밝은 환경에서는 90cm 또는 65cm 모두 포즈 인식률이나 카운트 안정성이 높지만, 어두운 환경까지 고려했을 때 90cm 위 보다 65cm 위가 조금 더 안정적임
    - DOWN 자세나 PLANK 자세 등 낮은 자세가 있어 너무 높은 위치보다 중간 위치가 더 적합한게 아닐까 싶음
- bottom 의 경우 가만히 서 있을 때 포즈 인식이 안됨과 더불어 운동 동작 중 landmark 가 많이 튀는 현상이 나타남

| 위치 \ 조명 | Light | Dark |
| --- | --- | --- |
| **90cm (삼각대)** | ✅ **5/5 (100%)** | ⚠️ **3/5 (60%)** |
| **65cm (삼각대)** | ✅ **5/5 (100%)** | ⚠️ **4/5 (80%)** |
| **bottom (바닥)** | ❌ **0/5 (0%)** | ❌ **0/5 (0%)** |

### 3.4.3 모바일 부가 기능 구현

‣ 

‣ 

‣ 

‣ 

- 운동 카운팅 로직이 동작하더라도, 실제 앱으로 사용하려면 누가 운동했는지 구분되고, 기록을 남길 수 있어야 함
- 또한 운동 중 오동작이 발생했을 때 사용자가 스스로 카운트를 초기화 하거나 기록을 관리할 수 있어야 함
- 이를 위해 서버 없이도 동작하는 최소 기능 기반의 부가 기능을 구현함

### 로그인/회원가입 (로컬 인증)

- 목적: 사용자를 구분하여 기록이 섞이는 문제 해결 → 개인별 기록 분리
- 흐름: 앱 실행 → AuthScreen(로그인/회원가입) → 로그인 성공 → HomeScreen
- 저장 방식: 서버 없이 SharedPreferences에 저장
    - `users`: 닉네임-비밀번호 목록
    - `current_user`: 현재 로그인 사용자

### 카운트 초기화

- 목적: 운동 중 카운트 꼬일 때 0부터 재시작, 단 READY/CALI 유지
- 동작: count=0 + FSM 초기 상태 복귀(스쿼트/푸쉬업 UP, 크런치 DOWN, 버피 STAND)

### 카운트 저장

- 목적: 현재 카운트를 1세트 기록으로 저장하고 저장 후 카운트를 초기화하여 바로 다음 세트 시작
- 저장 데이터: user / 날짜 / 운동 / reps / 세트시간(duration)
- 구조: SharedPreferences에 `logs_v1[user][date][exercise].sets[]` 형태로 누적

### 나의 기록

- 목적: 저장된 기록을 달력 기반으로 조회
- 동작: 날짜 선택 → 운동별 총합 리스트 → 운동 클릭 시 세트 상세(횟수/시간)

### 설정

- 기능: 운동 기록 초기화 / 회원 삭제(계정+기록)

## 4. 평가 및 결론

### 4.1 목표 대비 성과

본 프로젝트는 4가지 운동(스쿼트, 푸쉬업, 크런치, 버피테스트)에 대한 운동량 카운팅이 윈도우 데스크탑과 모바일 환경에서 구동 가능하도록 시스템을 구현하는 것을 목표로 하였으며, PoC(개념 검증) 단계에서 요구한 핵심 기능을 대부분 달성하였다.

- **PC + 모바일 구동**
    - PC(웹캠)로 로직 검증
    - 모바일(Flutter+Kotlin)로 이식 및 실시간 카운팅 구현
- **MediaPipe 기반 운동 카운팅 시스템 구현**
    - MediaPipe Pose 기반으로 관절 추론 및 스켈레톤 오버레이 구현
    - 4가지 운동(스쿼트, 푸쉬업, 크런치, 버피테스트) 카운팅 로직 구현
- **실사용을 위한 기능 확장**
    - 회원 인증, 세트 저장 및 기록 조회, 설정 등 개인별 카운팅을 위한 기능 확장
    - 촬영 위치 및 조명 실험 기반 권장 가이드 제공

### 4.2 한계점 및 개선 방향

프로토타입 단계에서 의미 있는 기능을 구현하였으나, 실제 상용 환경을 고려할 경우 다음과 같은 한계점이 확인되었다.

- **조명/노출 변화에 민감**
    - 조명 OFF 환경에서 Pose 인식률이 낮아져 카운팅 정확도가 저하
- **카메라 위치에 따른 성능 편차 큼**
    - 카메라가 바닥에 위치해 있는 등의 환경에서는 Pose 인식률이 급격히 저하
    - 운동별 최적 높이가 다를 수 있어 사용자가 직접 조정해야 하는 번거로움이 있을 수 있음
- **운동 자세 품질에 상관없이 카운트**
    - 잘못된 자세로 운동을 진행하여도 카운트 로직에 포함된다면 카운트 됨

### 4.3 종합 결론

본 프로젝트는 **MediaPipe Pose 랜드마크 출력 값을 이용하여 메트릭 설계 + 상태 머신 + 안정화(캘리/EMA/hold)를 직접 구성하여 4가지 맨몸운동을 카운트 하는 시스템을 구현**하였다. 

또한 PC에서의 가능성 검증에 그치지 않고, 실제 사용자가 운동을 하면서 사용할 수 있도록 Flutter+Kotlin 기반 모바일 앱으로 이식하여 실시간 동작과 인증/기록/설정까지 포함한 실제 사용 가능한 앱 형태로 완성하였다. 이 과정에서 촬영 위치 및 조명 실험을 수행하여 최적의 촬영 위치와 조명에 대한 가이드라인을 사용자에게 제공하였다.

하지만 실제 상용 운영을 위해서는 다양한 환경에 대한 안정성을 확보시키고 자세 품질 기반 카운팅 개선이 필요하다.