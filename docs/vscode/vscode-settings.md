# VS Code 설정 가이드

VS Code에서 프로젝트 개발을 위한 최적 설정을 제공합니다.

## 기본 설정

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true, // 파일 저장 시 자동 포맷팅
  "editor.defaultFormatter": "esbenp.prettier-vscode", // 기본 포매터로 Prettier 사용
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true // 저장 시 ESLint 자동 수정 적용
  },
  "eslint.validate": [
    "javascript", // JavaScript 파일에서 ESLint 검증
    "javascriptreact", // JSX 파일에서 ESLint 검증
    "typescript", // TypeScript 파일에서 ESLint 검증
    "typescriptreact" // TSX 파일에서 ESLint 검증
  ]
}
```

## TypeScript 설정

```json
// .vscode/settings.json에 추가
{
  "typescript.tsdk": "node_modules/typescript/lib", // 워크스페이스의 TypeScript 버전 사용
  "typescript.enablePromptUseWorkspaceTsdk": true, // 워크스페이스 TypeScript 사용 여부 묻기
  "typescript.preferences.importModuleSpecifier": "shortest", // import 경로를 가장 짧게 설정
  "typescript.updateImportsOnFileMove.enabled": "always" // 파일 이동 시 import 경로 자동 업데이트
}
```

## 워크스페이스별 설정

모노레포에서 각 패키지별로 다른 설정이 필요한 경우:

```json
// app/.vscode/settings.json
{
  "css.validate": false, // CSS 검증 비활성화 (Tailwind CSS 사용 시)
  "tailwindCSS.experimental.classRegex": [
    // clsx() 함수 내에서 Tailwind 클래스 인식
    ["clsx\\(([^)]*)\\)", "(?:'|\"|`)([^']*)(?:'|\"|`)"]
  ]
}
```

## 관련 문서

- [확장 프로그램](./vscode-extensions.md) - 추천 확장 프로그램 목록
- [디버깅 설정](./vscode-launch.md) - launch.json 설정
- [태스크 설정](./vscode-tasks.md) - tasks.json 설정
- [전체 IDE 설정](./README.md) - IDE 설정 개요
