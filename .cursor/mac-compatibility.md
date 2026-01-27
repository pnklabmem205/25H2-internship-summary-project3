# macOS 호환성 가이드

## 주요 차이점

### 1. 가상환경 활성화

**Windows:**
```bash
.\.venv\Scripts\activate
```

**macOS/Linux:**
```bash
source .venv/bin/activate
```

### 2. Python 명령어

**Windows:**
```bash
python --version
python -m venv .venv
```

**macOS/Linux:**
```bash
python3 --version
python3 -m venv .venv
```

### 3. 경로 예시

**Windows:**
- `C:\WS\flutter_projects`
- `D:\projects`

**macOS/Linux:**
- `~/projects`
- `/Users/username/projects`

### 4. Flutter Doctor 차이

**Windows:**
- `[✓] Visual Studio - develop Windows apps`

**macOS:**
- `[✓] Xcode - develop for iOS and macOS`

### 5. Gradle 실행

**Windows:**
```bash
gradlew.bat clean
```

**macOS/Linux:**
```bash
./gradlew clean
```

### 6. 카메라 권한 (macOS)

macOS에서는 터미널 앱에 카메라 권한을 부여해야 할 수 있습니다:

1. 시스템 설정 → 보안 및 개인정보 보호 → 카메라
2. 터미널 앱 체크
3. 또는 코드 실행 시 권한 요청 팝업에서 허용

### 7. PowerShell 명령어 (Windows 전용)

다음 명령어는 Windows PowerShell 전용입니다:
- `Select-String`
- `ForEach-Object`
- `Out-File`
- `Get-ExecutionPolicy`
- `Set-ExecutionPolicy`

**macOS/Linux 대체:**
```bash
# PowerShell 파이프라인 대신
flutter run | grep "POSECSV\|EXLOG" > log_file.log
```

## 호환성 확인

### PC 버전
- ✅ Python 3.10.8: macOS/Linux 지원
- ✅ MediaPipe 0.10.21: macOS/Linux 지원
- ✅ OpenCV 4.11.0.86: macOS/Linux 지원
- ⚠️ 웹캠: macOS에서 카메라 권한 필요

### 모바일 버전
- ✅ Flutter: 크로스 플랫폼 (Windows/macOS/Linux 모두 지원)
- ✅ Android 개발: macOS에서도 가능
- ✅ 모든 명령어: 크로스 플랫폼 호환

## 주의사항

1. **가상환경 활성화**: macOS는 `source .venv/bin/activate` 사용
2. **Python 명령어**: macOS는 `python3` 사용 권장
3. **카메라 권한**: macOS에서 터미널 앱 권한 확인 필요
4. **경로 구분자**: macOS/Linux는 `/` 사용 (Windows는 `\`)
