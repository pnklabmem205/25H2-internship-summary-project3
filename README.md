# 프로젝트3 인수인계 가이드

프로젝트3 (MediaPipe 기반 맨몸운동 자동 카운터 시스템) 인수인계를 위한 가이드 문서 모음입니다.

## 📚 주요 문서

### 설치 및 운영 가이드
- **[프로젝트3_설치_운영_가이드.md](./프로젝트3_설치_운영_가이드.md)**: 전체 설치 및 운영 가이드
- **[Python_3.10.8_설치_가이드.md](./Python_3.10.8_설치_가이드.md)**: Python 3.10.8 설치 가이드 (필수)
- **[GIT_사용_가이드.md](./GIT_사용_가이드.md)**: Git 저장소 사용 방법 (노트북에서 테스트용)

### 점검 및 체크리스트
- **[프로젝트3_점검_체크리스트.md](./프로젝트3_점검_체크리스트.md)**: 프로젝트 점검 항목
- **[인수인계_가이드_및_체크리스트.md](./인수인계_가이드_및_체크리스트.md)**: 종합 인수인계 가이드

### 프로젝트 정보
- **[project3info.md](./project3info.md)**: 프로젝트 개요 및 기술 문서

### Cursor 설정 (AI 개발 도우미용)
- **[.cursor/rules.md](./.cursor/rules.md)**: 프로젝트 규칙
- **[.cursor/tech-stack.md](./.cursor/tech-stack.md)**: 기술 스택 정보
- **[.cursor/coding-patterns.md](./.cursor/coding-patterns.md)**: 코딩 패턴
- **[.cursor/project-context.md](./.cursor/project-context.md)**: 프로젝트 컨텍스트
- **[.cursor/mac-compatibility.md](./.cursor/mac-compatibility.md)**: macOS 호환성 가이드

## 🚀 빠른 시작

### 1. 프로젝트 이해
1. [프로젝트 개요](./프로젝트3_설치_운영_가이드.md#1-프로젝트-개요) 읽기
2. [빠른 시작 가이드](./프로젝트3_설치_운영_가이드.md#2-빠른-시작---어떤-버전을-설치해야-하나) 확인

### 2. 설치 (권장 순서)
1. **모바일 버전 설치** (최우선): [모바일 버전 설치 가이드](./프로젝트3_설치_운영_가이드.md#모바일-버전-설치-및-실행)
2. **PC 버전 설치** (선택): [PC 버전 설치 가이드](./프로젝트3_설치_운영_가이드.md#pc-버전-설치-및-실행)
3. **HEADPOSE 설치** (선택): [HEADPOSE 설치 가이드](./프로젝트3_설치_운영_가이드.md#headpose-설치-및-실행)

### 3. 순서대로 따라하기 (권장)
[순서대로 따라하기 가이드](./프로젝트3_설치_운영_가이드.md#순서대로-따라하기-가이드)를 따라 단계별로 학습하세요.

## ⚠️ 중요 사항

### Python 버전
- **Python 3.10.8 필수**: MediaPipe 0.10.21 호환을 위해 필요
- Python 3.13+에서는 MediaPipe 0.10.21 설치 불가
- [Python 3.10.8 설치 가이드](./Python_3.10.8_설치_가이드.md) 참고

### 저장소 구조
- **workout-counter-pc**: 알고리즘 검증용 (Python) - 선택
- **workout-counter-mobile**: 실제 사용용 앱 (Flutter) - 필수 ⭐
- **HEADPOSE**: 별도 실험 프로젝트 (참고용)

## 🔧 문제 해결

문제가 발생하면 [트러블슈팅 섹션](./프로젝트3_설치_운영_가이드.md#6-트러블슈팅)을 먼저 확인하세요.

## 📖 참고 자료

### 저장소
- PC 버전: https://github.com/pnkmem432/workout-counter-pc
- 모바일 버전: https://github.com/pnkmem432/workout-counter-mobile
- HEADPOSE: https://github.com/pnkmem432/HEADPOSE
- 구글 드라이브: https://drive.google.com/drive/folders/1r3qVubDVXiEYmKXj-QL-3Q1kLxstGouo?hl=ko

### 기술 문서
- MediaPipe 공식 문서: https://mediapipe.readthedocs.io/
- Flutter 공식 문서: https://flutter.dev/docs

---

**작성일**: 2026-01-27  
**목적**: 프로젝트3 인수인계를 위한 종합 가이드
