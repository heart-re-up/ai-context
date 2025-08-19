# IDE 및 에디터 설정 가이드

프로젝트 개발을 위한 IDE 및 에디터 설정에 대한 전체 가이드입니다.

## 개요

이 프로젝트는 VS Code를 주요 IDE로 사용하며, 최적화된 개발 환경을 위한 설정을 제공합니다.

## 지원 IDE

### VS Code (권장)

- 전체 기능 지원
- 확장 프로그램 및 설정 최적화
- 디버깅 및 태스크 설정 포함

### IntelliJ IDEA / WebStorm

- 기본적인 프로젝트 설정은 자동으로 인식됩니다
- 별도 설정 파일 관리 불필요

## 상세 설정 문서

### 📁 VS Code 설정

- **[기본 설정](./vscode-settings.md)** - VS Code 기본 설정 및 TypeScript 설정
- **[확장 프로그램](./vscode-extensions.md)** - 추천 확장 프로그램 목록
- **[디버깅 설정](./vscode-launch.md)** - launch.json 설정 및 디버깅 방법
- **[태스크 설정](./vscode-tasks.md)** - tasks.json 설정 및 빌드/테스트 태스크

### 📁 프로젝트 설정

- **[Git 무시 설정](./project-gitignore.md)** - .gitignore 설정 및 버전 관리 제외 파일

## 빠른 시작

1. **VS Code 설치** 및 권장 확장 프로그램 설치
2. **워크스페이스 설정** 적용 (`.vscode/settings.json`)
3. **프로젝트 의존성** 설치 (`pnpm install`)
4. **개발 서버** 시작 (`pnpm dev`)

자세한 내용은 각 상세 문서를 참조하세요.
