# VS Code 확장 프로그램 가이드

VS Code에서 프로젝트 개발을 위한 추천 확장 프로그램 목록입니다.

## 추천 확장 프로그램

```json
// .vscode/extensions.json
{
  "recommendations": [
    "esbenp.prettier-vscode", // 코드 포맷터 (Prettier)
    "dbaeumer.vscode-eslint", // JavaScript/TypeScript 린터 (ESLint)
    "bradlc.vscode-tailwindcss", // Tailwind CSS 인텔리센스 및 자동완성
    "burkeholland.simple-react-snippets", // React 기본 스니펫 모음
    "dsznajder.es7-react-js-snippets", // React/Redux/ES7+ 스니펫 모음
    "formulahendry.auto-rename-tag", // HTML/JSX 태그 자동 이름 변경
    "formulahendry.auto-close-tag", // HTML/JSX 태그 자동 닫기
    "christian-kohler.path-intellisense", // 파일 경로 자동완성
    "editorconfig.editorconfig", // EditorConfig 지원
    "yoavbls.pretty-ts-errors", // TypeScript 에러 메시지 개선
    "vitest.explorer" // Vitest 테스트 러너 및 익스플로러
  ]
}
```

## 확장 프로그램 상세 설명

### 🎨 코드 포맷팅 및 린팅

- **Prettier** (`esbenp.prettier-vscode`): 코드 자동 포맷팅
- **ESLint** (`dbaeumer.vscode-eslint`): JavaScript/TypeScript 코드 품질 검사

### 🎯 React 개발

- **Simple React Snippets** (`burkeholland.simple-react-snippets`): 기본 React 스니펫
- **ES7+ React/Redux/React-Native snippets** (`dsznajder.es7-react-js-snippets`): 확장된 React 스니펫

### 🏷️ HTML/JSX 태그 관리

- **Auto Rename Tag** (`formulahendry.auto-rename-tag`): 태그 이름 자동 변경
- **Auto Close Tag** (`formulahendry.auto-close-tag`): 태그 자동 닫기

### 🎨 CSS 및 스타일링

- **Tailwind CSS IntelliSense** (`bradlc.vscode-tailwindcss`): Tailwind CSS 자동완성 및 클래스 제안

### 🔧 개발 도구

- **Path Intellisense** (`christian-kohler.path-intellisense`): 파일 경로 자동완성
- **EditorConfig** (`editorconfig.editorconfig`): 에디터 설정 통일
- **Pretty TypeScript Errors** (`yoavbls.pretty-ts-errors`): TypeScript 에러 메시지 개선
- **Vitest** (`vitest.explorer`): 테스트 러너 및 익스플로러

## 설치 방법

### 일반 환경 (인터넷 연결 가능)

1. **자동 설치**: VS Code에서 워크스페이스 열 때 추천 확장 프로그램 설치 알림이 표시됩니다.
2. **수동 설치**: `Ctrl+Shift+X`로 확장 프로그램 패널 열고 각각 검색하여 설치
3. **명령어로 설치**:
   ```bash
   code --install-extension esbenp.prettier-vscode
   code --install-extension dbaeumer.vscode-eslint
   # ... 기타 확장 프로그램들
   ```

### 격리된 환경 (오프라인 설치)

인터넷 접속이 제한된 격리망이나 보안 환경에서는 `.vsix` 파일을 사용하여 확장 프로그램을 설치할 수 있습니다.

#### 1단계: VSIX 파일 다운로드

1. [Visual Studio Marketplace](https://marketplace.visualstudio.com/vscode)에 접속
2. 필요한 확장 프로그램 검색 (예: "Prettier - Code formatter")
3. 확장 프로그램 페이지에서 **"Download Extension"** 버튼 클릭
4. `.vsix` 확장자의 설치 파일 다운로드

#### 2단계: 격리 환경으로 파일 전송

- **입고 프로세스**: 조직의 보안 정책에 따른 파일 입고 절차 수행
- **전송 프로그램**: 승인된 파일 전송 도구 사용
- **물리적 매체**: USB, CD 등 허용된 저장 매체 활용

#### 3단계: VSIX 파일 설치

**방법 1: UI를 통한 설치 (VS Code)**

1. VS Code 실행
2. 확장 프로그램 패널 (`Ctrl+Shift+X`) 열기
3. 상단 `...` 메뉴 클릭
4. **"Install from VSIX..."** 선택
5. 전송된 `.vsix` 파일 선택하여 설치

**방법 2: 드래그 앤 드롭 설치**

- `.vsix` 파일을 VS Code/Cursor 창으로 드래그하여 놓기 (UI 환경에서만 가능)

**방법 3: 명령어를 통한 설치 (VS Code & Cursor 공통)**

```bash
# VS Code 명령어
code --install-extension /path/to/extension.vsix

# Cursor 명령어 (cursor 명령이 PATH에 등록된 경우)
cursor --install-extension /path/to/extension.vsix
```

**방법 4: 명령 팔레트를 통한 설치 (Cursor & VS Code 공통)**

1. 명령 팔레트 열기 (`Ctrl+Shift+P` 또는 `Cmd+Shift+P`)
2. `install from` 또는 `Extensions: Install from VSIX...` 검색
3. **"Extensions: Install from VSIX..."** 선택
4. 전송된 `.vsix` 파일 선택하여 설치

**방법 5: 터미널을 통한 설치 (대안 방법)**

1. 터미널 열기 (`Ctrl + ~` 또는 `View` > `Terminal`)
2. 다음 명령어 실행:
   ```bash
   code --install-extension /path/to/extension.vsix
   ```
3. 에디터 재시작하여 확장 프로그램 로드 확인

> **참고**: Cursor와 VS Code 모두 명령 팔레트(`Ctrl+Shift+P` / `Cmd+Shift+P`)를 통해 "Extensions: Install from VSIX..." 명령을 제공합니다. 이는 확장 프로그램 패널에서 해당 메뉴를 찾을 수 없을 때 유용한 대안입니다.

#### 주의사항

- 확장 프로그램의 종속성도 함께 다운로드해야 할 수 있습니다
- 버전 호환성을 확인하여 적절한 버전의 `.vsix` 파일을 선택하세요
- 조직의 보안 정책을 준수하여 파일 전송 및 설치를 진행하세요

## 관련 문서

- [VS Code 설정](./vscode-settings.md) - 기본 설정 및 TypeScript 설정
- [디버깅 설정](./vscode-launch.md) - launch.json 설정
- [태스크 설정](./vscode-tasks.md) - tasks.json 설정
