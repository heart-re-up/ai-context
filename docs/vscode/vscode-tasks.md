# VS Code 태스크 설정

VS Code에서 프로젝트 빌드 및 테스트를 위한 태스크 설정입니다.

## 태스크 설정

```json
// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "dev", // 태스크 이름
      "type": "npm", // npm 스크립트 실행
      "script": "dev", // package.json의 dev 스크립트 실행
      "problemMatcher": [], // 문제 매처 비활성화
      "isBackground": true, // 백그라운드 태스크로 실행
      "presentation": {
        "reveal": "always", // 항상 터미널 표시
        "panel": "new" // 새 터미널 패널에서 실행
      },
      "group": {
        "kind": "build", // 빌드 그룹에 속함
        "isDefault": true // 기본 빌드 태스크로 설정
      }
    },
    {
      "label": "test", // 태스크 이름
      "type": "npm", // npm 스크립트 실행
      "script": "test", // package.json의 test 스크립트 실행
      "problemMatcher": [], // 문제 매처 비활성화
      "presentation": {
        "reveal": "always", // 항상 터미널 표시
        "panel": "new" // 새 터미널 패널에서 실행
      },
      "group": {
        "kind": "test", // 테스트 그룹에 속함
        "isDefault": true // 기본 테스트 태스크로 설정
      }
    }
  ]
}
```

## 사용법

1. **개발 서버 시작**: `Ctrl+Shift+P` → `Tasks: Run Task` → `dev`
2. **테스트 실행**: `Ctrl+Shift+P` → `Tasks: Run Task` → `test`
3. **기본 빌드 태스크**: `Ctrl+Shift+B` (dev 태스크 실행)
4. **기본 테스트 태스크**: `Ctrl+Shift+P` → `Tasks: Run Test Task`

## 커스텀 태스크 추가

추가적인 태스크가 필요한 경우:

```json
{
  "label": "build", // 빌드 태스크
  "type": "npm",
  "script": "build",
  "group": "build",
  "problemMatcher": ["$tsc"] // TypeScript 컴파일 에러 감지
},
{
  "label": "lint", // 린트 태스크
  "type": "npm",
  "script": "lint",
  "group": "build",
  "problemMatcher": ["$eslint-stylish"] // ESLint 에러 감지
}
```

## 관련 문서

- [VS Code 설정](./vscode-settings.md) - 기본 VS Code 설정
- [확장 프로그램](./vscode-extensions.md) - 추천 확장 프로그램
- [디버깅 설정](./vscode-launch.md) - launch.json 설정
