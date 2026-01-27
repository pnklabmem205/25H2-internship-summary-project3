# Python 3.10.8 설치 가이드

프로젝트3는 MediaPipe 0.10.21을 사용하며, 이 버전은 Python 3.10.8에서 가장 안정적으로 작동합니다.

## Windows 설치 방법

### 1. Python 3.10.8 다운로드

1. Python 공식 사이트 방문: https://www.python.org/downloads/release/python-3108/
2. "Windows installer (64-bit)" 다운로드
   - 파일명: `python-3.10.8-amd64.exe`

### 2. 설치

1. 다운로드한 설치 파일 실행
2. 설치 옵션:
   - ✅ **"Add Python 3.10 to PATH"** 체크 (선택사항)
     - 다른 Python 버전과 충돌할 수 있으므로 주의
     - 체크하지 않아도 `py -3.10` 명령어로 사용 가능
   - ✅ **"Install for all users"** 체크 (선택사항)
   - "Customize installation" 클릭하여 설치 경로 확인 가능
3. "Install Now" 클릭하여 설치 진행

### 3. 설치 확인

**PowerShell 또는 CMD에서:**
```powershell
# py launcher 사용 (권장)
py -3.10 --version
# 예상 출력: Python 3.10.8

# 또는 전체 경로 사용
C:\Users\사용자명\AppData\Local\Programs\Python\Python310\python.exe --version
```

### 4. 가상환경 생성

**workout-counter-pc 폴더에서:**
```powershell
# 1. 기존 가상환경 비활성화 (활성화되어 있다면)
deactivate

# 2. 기존 가상환경 폴더 삭제
Remove-Item -Recurse -Force .venv

# 3. Python 3.10.8로 새 가상환경 생성
py -3.10 -m venv .venv

# 4. 가상환경 활성화
.\.venv\Scripts\activate

# 5. Python 버전 확인 (3.10.8이어야 함)
python --version
```

**가상환경 비활성화 방법:**
```powershell
# 가상환경이 활성화되어 있을 때 (프롬프트에 (.venv) 표시)
deactivate

# 비활성화 후 프롬프트에서 (.venv)가 사라짐
```

## macOS 설치 방법

### Homebrew 사용 (권장)

```bash
# Homebrew로 Python 3.10.8 설치
brew install python@3.10

# 가상환경 생성
python3.10 -m venv .venv

# 가상환경 활성화
source .venv/bin/activate

# 버전 확인
python --version
```

### 공식 설치 파일 사용

1. Python 공식 사이트: https://www.python.org/downloads/release/python-3108/
2. "macOS 64-bit universal2 installer" 다운로드
3. 설치 파일 실행하여 설치

## Linux 설치 방법

### Ubuntu/Debian

```bash
# 저장소 업데이트
sudo apt-get update

# Python 3.10.8 설치
sudo apt-get install python3.10 python3.10-venv python3.10-pip

# 가상환경 생성
python3.10 -m venv .venv

# 가상환경 활성화
source .venv/bin/activate

# 버전 확인
python --version
```

## 여러 Python 버전 관리

### Windows: py launcher 사용

Windows에는 `py` launcher가 포함되어 있어 여러 Python 버전을 쉽게 관리할 수 있습니다:

```powershell
# 설치된 Python 버전 확인
py --list

# 특정 버전으로 가상환경 생성
py -3.10 -m venv .venv310
py -3.13 -m venv .venv313

# 특정 버전으로 스크립트 실행
py -3.10 script.py
py -3.13 script.py
```

### macOS/Linux: pyenv 사용 (선택사항)

```bash
# pyenv 설치 (Homebrew)
brew install pyenv

# Python 3.10.8 설치
pyenv install 3.10.8

# 프로젝트 폴더에서 Python 3.10.8 사용
cd workout-counter-pc
pyenv local 3.10.8

# 가상환경 생성
python -m venv .venv
```

## 문제 해결

### 문제: `py -3.10` 명령어가 작동하지 않음

**해결:**
1. Python 3.10.8이 제대로 설치되었는지 확인
2. 전체 경로 사용:
   ```powershell
   C:\Users\사용자명\AppData\Local\Programs\Python\Python310\python.exe -m venv .venv
   ```

### 문제: 가상환경 생성 후에도 Python 3.13이 사용됨

**해결:**
1. 가상환경이 제대로 활성화되었는지 확인 (`(.venv)` 표시 확인)
2. 가상환경 내부 Python 확인:
   ```powershell
   .\.venv\Scripts\python.exe --version
   ```
3. 문제가 계속되면 가상환경 재생성:
   ```powershell
   Remove-Item -Recurse -Force .venv
   py -3.10 -m venv .venv
   ```

### 문제: MediaPipe 0.10.21 설치 실패

**해결:**
1. Python 버전 확인: `python --version` (3.10.8이어야 함)
2. pip 업그레이드: `python -m pip install --upgrade pip`
3. MediaPipe 설치: `pip install mediapipe==0.10.21`

## 확인 체크리스트

설치 완료 후 확인:

- [ ] `py -3.10 --version` 또는 `python3.10 --version` 실행 시 Python 3.10.8 출력
- [ ] 가상환경 생성: `py -3.10 -m venv .venv` 성공
- [ ] 가상환경 활성화 후 `python --version` 실행 시 Python 3.10.8 출력
- [ ] `pip install mediapipe==0.10.21` 성공
- [ ] `python test_pose_webcam.py` 실행 성공

---

**참고**: Python 3.10.8은 프로젝트3의 권장 버전입니다.  
다른 Python 버전을 사용하면 MediaPipe 호환성 문제가 발생할 수 있습니다.
