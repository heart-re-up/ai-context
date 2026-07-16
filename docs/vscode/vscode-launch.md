# VS Code 디버깅 설정

VS Code에서 React 애플리케이션과 테스트를 디버깅하기 위한 설정입니다.

## 디버깅 설정

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome", // Chrome 브라우저에서 디버깅
      "request": "launch", // 새 인스턴스 시작
      "name": "Launch Chrome against localhost",
      "url": "http://localhost:5173", // Vite 개발 서버 URL
      "webRoot": "${workspaceFolder}/app/src", // 웹 루트 디렉토리
      "sourceMaps": true // 소스맵 사용
    },
    {
      "type": "node", // Node.js 환경에서 디버깅
      "request": "launch", // 새 인스턴스 시작
      "name": "Debug Vitest Tests",
      "skipFiles": ["<node_internals>/**"], // Node.js 내부 파일 제외
      "program": "${workspaceFolder}/node_modules/vitest/vitest.mjs", // Vitest 실행 파일
      "args": ["run", "${file}"], // 현재 파일만 테스트
      "smartStep": true, // 스마트 스테핑 활성화
      "console": "integratedTerminal" // 통합 터미널 사용
    }
  ]
}
```

## 사용법

1. **React 앱 디버깅**:

   - 개발 서버(`pnpm dev`) 실행 후 `Launch Chrome against localhost` 설정 사용
   - 브레이크포인트 설정 후 F5로 디버깅 시작

2. **테스트 디버깅**:
   - 테스트 파일에서 `Debug Vitest Tests` 설정 사용
   - 테스트 코드에 브레이크포인트 설정 후 디버깅 실행

## 관련 문서

- [VS Code 설정](./vscode-settings.md) - 기본 VS Code 설정
- [확장 프로그램](./vscode-extensions.md) - 추천 확장 프로그램
- [태스크 설정](./vscode-tasks.md) - tasks.json 설정
