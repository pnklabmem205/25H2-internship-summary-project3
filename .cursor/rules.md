# 프로젝트3 인수인계 Rules

## 프로젝트 개요

**프로젝트명**: MediaPipe 기반 맨몸운동 자동 카운터 시스템

**저장소 구조**:
- `workout-counter-pc`: 알고리즘 검증용 (Python) - 선택
- `workout-counter-mobile`: 실제 사용용 앱 (Flutter) - 필수 ⭐

**개발 흐름**: PC 버전 검증 → 모바일 버전 이식

## 기술 스택

### PC 버전
- Python 3.10.8 (필수)
- MediaPipe 0.10.21
- OpenCV 4.11.0.86
- NumPy

### 모바일 버전
- Flutter (최신 안정 버전)
- Dart
- Android Kotlin
- MediaPipe PoseLandmarker
- SharedPreferences

## 코딩 규칙

### MediaPipe 사용 패턴
```python
# PC 버전: BGR → RGB 변환 필수
img_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
results = pose.process(img_rgb)
```

### 카운팅 로직 구조
- READY → CALI → COUNT 단계 구조 필수
- 상태 머신(FSM) 기반 카운팅
- EMA로 노이즈 완화
- Visibility 체크 필수

### 운동별 메트릭
- 스쿼트: 힙 하강량 정규화 (`depth_pct = drop / ref_len0`)
- 푸쉬업: 어깨 하강량 정규화 (`d_norm = (mid_sh_y - sh0_y) / sw0`)
- 크런치: 상체 접힘 각도 (3D 좌표 기반)
- 버피: 상태 머신 기반 (STAND → DOWN → PLANK → SQUAT → STAND)

## 파일 구조

### PC 버전
- `v0.4/`: 최종 버전 (권장)
- `main_<운동명>.py`: 실행 파일

### 모바일 버전
- `lib/counters/`: 카운터 로직
- `lib/screens/`: 화면
- `lib/services/`: 서비스
- `android/app/src/main/kotlin/MainActivity.kt`: MediaPipe 처리

## 주의사항

- PC 버전: Python 3.10.8 필수, 라이브러리 버전 고정
- 모바일 버전: `extras` 브랜치가 최종 통합 버전
- MediaPipe는 Flutter에서 직접 실행 불가 → Kotlin 네이티브 사용
- 카운팅 로직 수정 시 PC 버전에서 먼저 검증 후 모바일로 이식

## 참고 문서

- `project3info.md`: 프로젝트 개요
- `프로젝트3_설치_운영_가이드.md`: 설치 가이드
- `프로젝트3_점검_체크리스트.md`: 점검 체크리스트
- `p3_docs/`: 데일리 문서
