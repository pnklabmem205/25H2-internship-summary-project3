# 프로젝트3 코딩 패턴

## MediaPipe 사용 패턴

### PC 버전 (Python)
```python
import cv2
import mediapipe as mp

# 1. 카메라 프레임 읽기
cap = cv2.VideoCapture(0)
success, frame = cap.read()

# 2. BGR → RGB 변환 (필수)
img_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)

# 3. MediaPipe Pose 처리
mp_pose = mp.solutions.pose
with mp_pose.Pose(static_image_mode=False) as pose:
    results = pose.process(img_rgb)
    
# 4. 랜드마크 활용
if results.pose_landmarks:
    landmarks = results.pose_landmarks.landmark
    # 33개 랜드마크 좌표 사용
```

### 모바일 버전 (Flutter ↔ Kotlin)

**Flutter → Kotlin (프레임 전달)**:
```dart
await platform.invokeMethod('processFrameNv21', {
  'bytes': frameBytes,
  'width': width,
  'height': height,
  'isFront': true,
  'timestamp': timestamp,
});
```

**Kotlin → Flutter (랜드마크 스트리밍)**:
```dart
EventChannel('poseLandmarks')
  .receiveBroadcastStream()
  .listen((landmarks) {
    // 랜드마크 처리
  });
```

## 카운팅 로직 패턴

### 공통 구조
1. **READY 단계**: 포즈 인식 안정화 (연속 N프레임)
2. **CALI 단계**: 기준 자세 학습 (3초 데이터 수집)
3. **COUNT 단계**: 상태 머신 기반 카운팅

### 안정화 기법

**EMA (지수 이동 평균)**:
```
ema = (1 - alpha) * prev_ema + alpha * new_value
alpha = 0.2 (일반적)
```

**Visibility 체크**:
```python
vis_min = min(
    left_shoulder.visibility,
    right_shoulder.visibility,
    left_hip.visibility,
    right_hip.visibility
)
if vis_min >= 0.5:  # 임계값
    # 계산 진행
```

**Hold 프레임**:
- 조건을 N프레임 연속 만족해야 상태 전환
- 예: `hold_frames = 5`

## 운동별 카운팅 방식

### 스쿼트
- 메트릭: `depth_pct = drop / ref_len0`
- 상태 전환: `UP → DOWN (≥0.25) → UP (≤0.12) → count+1`

### 푸쉬업
- 메트릭: `d_norm = (mid_sh_y - sh0_y) / sw0`
- 상태 전환: `UP → DOWN (≥0.30) → UP (≤0.25) → count+1`

### 크런치
- 메트릭: `flex = 180 - raw_angle` (3D 좌표)
- 상태 전환: `DOWN → UP (swing≥12°) → DOWN → count+1`

### 버피테스트
- 상태 전환: `STAND → DOWN → PLANK → SQUAT → STAND → count+1`
