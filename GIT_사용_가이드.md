# Git 저장소 사용 가이드

프로젝트3 인수인계 가이드를 Git으로 관리하여 다른 환경(노트북 등)에서도 사용할 수 있도록 설정되었습니다.

## 현재 상태

- ✅ Git 저장소 초기화 완료
- ✅ 주요 가이드 문서 커밋 완료
- ✅ .gitignore 설정 완료
- ✅ .gitattributes 설정 완료 (한글 파일명 지원)

## 커밋된 파일

### 가이드 문서
- `README.md`: 프로젝트 개요 및 빠른 시작
- `프로젝트3_설치_운영_가이드.md`: 전체 설치 및 운영 가이드
- `Python_3.10.8_설치_가이드.md`: Python 3.10.8 설치 가이드
- `프로젝트3_점검_체크리스트.md`: 프로젝트 점검 항목
- `인수인계_가이드_및_체크리스트.md`: 종합 인수인계 가이드

### 프로젝트 정보
- `project3info.md`: 프로젝트 개요 및 기술 문서
- `project1info.md`, `project2info.md`: 다른 프로젝트 정보

### 데일리 문서
- `p3_docs/`: 프로젝트3 개발 일지 (20개 파일)

### Cursor 설정
- `.cursor/`: AI 개발 도우미 설정 파일

## 노트북에서 사용하기

### 방법 1: 원격 저장소에 푸시 후 클론 (권장)

**1. GitHub에 새 저장소 생성**
- GitHub에서 새 저장소 생성 (예: `project3-handover-guide`)

**2. 원격 저장소 추가 및 푸시**
```bash
cd D:\takeover2601

# 원격 저장소 추가
git remote add origin https://github.com/사용자명/project3-handover-guide.git

# 브랜치 이름을 main으로 변경 (선택사항)
git branch -M main

# 푸시
git push -u origin main
```

**3. 노트북에서 클론**
```bash
git clone https://github.com/사용자명/project3-handover-guide.git
cd project3-handover-guide
```

### 방법 2: USB/네트워크로 복사

**1. 현재 저장소를 압축**
```bash
cd D:\takeover2601
# .git 폴더 포함하여 전체 복사
```

**2. 노트북으로 복사 후 사용**
```bash
# 노트북에서
cd 복사한_폴더
git status  # 저장소 상태 확인
```

### 방법 3: Git Bundle 사용

**1. Bundle 파일 생성**
```bash
cd D:\takeover2601
git bundle create project3-handover-guide.bundle --all
```

**2. 노트북에서 복원**
```bash
# 노트북에서
git clone project3-handover-guide.bundle project3-handover-guide
cd project3-handover-guide
```

## Git 명령어 참고

### 기본 명령어
```bash
# 상태 확인
git status

# 변경사항 확인
git diff

# 커밋 내역 확인
git log --oneline

# 파일 추가
git add 파일명

# 커밋
git commit -m "커밋 메시지"

# 원격 저장소에 푸시
git push origin main
```

### 브랜치 관리
```bash
# 브랜치 목록
git branch

# 새 브랜치 생성
git checkout -b 새브랜치명

# 브랜치 전환
git checkout 브랜치명
```

## 주의사항

### 제외된 파일들
`.gitignore`에 의해 다음 파일들은 커밋되지 않습니다:
- `.venv/`: Python 가상환경 (용량이 큼)
- `*.pt`: MediaPipe 모델 파일 (용량이 큼)
- `p3/workout-counter-pc/.venv/`: PC 버전 가상환경

### 포함된 파일들
- 모든 가이드 문서 (.md 파일)
- p3_docs/ 폴더의 데일리 문서
- .cursor/ 폴더의 설정 파일

## 업데이트 방법

가이드를 수정한 후:

```bash
# 변경사항 확인
git status

# 변경된 파일 추가
git add 수정한파일.md

# 커밋
git commit -m "가이드 업데이트: 변경 내용 요약"

# 원격 저장소에 푸시 (원격 저장소가 설정된 경우)
git push origin main
```

## 문제 해결

### 한글 파일명 문제
- `.gitattributes` 파일로 한글 파일명 지원 설정됨
- `git config core.quotepath false` 설정으로 한글 표시 개선

### 줄바꿈 문제
- `.gitattributes`에서 `eol=lf` 설정으로 통일
- Windows/Mac/Linux 간 호환성 확보

---

**작성일**: 2026-01-27  
**목적**: Git 저장소 사용 방법 안내
