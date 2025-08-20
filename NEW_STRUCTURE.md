# AI Context 개선된 구조

## 현재 문제점
1. 정보가 여러 파일에 분산되어 있음
2. 깊은 디렉토리 구조로 탐색이 비효율적
3. 중복된 내용이 많음

## 제안하는 새로운 구조

```
ai-context/
├── README.md                    # 간단한 소개와 사용법
├── eslint-config.md            # 모든 ESLint 설정 통합
├── typescript-config.md        # 모든 TypeScript 설정 통합
├── build-tools.md              # Vite, Vitest 설정 통합
├── code-quality.md             # Prettier, EditorConfig 통합
├── monorepo-setup.md           # 모노레포 전체 가이드 통합
├── development-environment.md  # VSCode, Node.js 환경 설정
├── deployment.md               # Docker, CI/CD 통합
└── automation.md               # 자동화 도구 통합
```

## 통합 계획

### 1. ESLint 통합 (eslint-config.md)
- quality/eslint.md
- quality/monorepo-config.md의 ESLint 부분
- monorepo/config.md의 ESLint 부분

### 2. TypeScript 통합 (typescript-config.md)
- typescript.md
- quality/monorepo-config.md의 TypeScript 부분
- monorepo/config.md의 TypeScript 부분

### 3. 빌드 도구 통합 (build-tools.md)
- vite/ 디렉토리 전체
- vitest/ 디렉토리 전체

### 4. 코드 품질 통합 (code-quality.md)
- quality/prettier.md
- quality/editorconfig.md
- quality/scripts.md

### 5. 모노레포 통합 (monorepo-setup.md)
- monorepo/ 디렉토리 전체를 하나로

### 6. 개발 환경 통합 (development-environment.md)
- vscode/ 디렉토리
- node/ 디렉토리

### 7. 배포 통합 (deployment.md)
- docker/ 디렉토리
- cicd/ 디렉토리

### 8. 자동화 통합 (automation.md)
- automation/ 디렉토리

## 장점
1. AI가 관련 정보를 한 파일에서 모두 찾을 수 있음
2. 중복 제거로 파일 크기 감소
3. 더 빠른 컨텍스트 로딩
4. 유지보수 용이