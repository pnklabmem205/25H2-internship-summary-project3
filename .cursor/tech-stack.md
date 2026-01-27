# 프로젝트3 기술 스택

## PC 버전 (workout-counter-pc)

**언어**: Python 3.10.8

**핵심 라이브러리**:
- MediaPipe 0.10.21
- OpenCV 4.11.0.86
- NumPy

**버전 구조**:
- v0.2: 측면 기준
- v0.3: 정면 기준
- v0.4: 최종 버전 (권장)

## 모바일 버전 (workout-counter-mobile)

**프레임워크**: Flutter + Dart

**네이티브**: Android Kotlin

**MediaPipe**: PoseLandmarker (Android 네이티브)

**저장소**: SharedPreferences

**브랜치**:
- squat, pushup, crunch, burpee: 운동별 구현
- extras: 최종 통합 버전 (권장)

## MediaPipe Pose

- 33개 관절 랜드마크
- 2D 좌표: 화면 기준 (0~1)
- 3D 좌표: 공간 기준 (각도 계산)
- Visibility: 관절 가시성 (0~1)
