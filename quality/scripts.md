# 코드 품질 스크립트 및 자동화 가이드

코드 품질 도구들을 통합하여 사용하는 스크립트와 자동화 설정을 다룹니다.

## 기본 스크립트 (package.json)

```json
{
  "scripts": {
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "format": "prettier --check \"src/**/*.{ts,tsx,json,md}\"",
    "format:fix": "prettier --write \"src/**/*.{ts,tsx,json,md}\"",
    "quality": "pnpm lint && pnpm format",
    "quality:fix": "pnpm lint:fix && pnpm format:fix"
  }
}
```

## 고급 스크립트

### 성능 최적화된 스크립트

```json
{
  "scripts": {
    "lint": "eslint . --cache --cache-location .eslintcache --ext ts,tsx",
    "lint:fix": "eslint . --cache --fix --ext ts,tsx",
    "lint:ci": "eslint . --max-warnings 0 --format json --output-file eslint-report.json",
    "format": "prettier --check --cache \"src/**/*.{ts,tsx,json,md,css,scss}\"",
    "format:fix": "prettier --write --cache \"src/**/*.{ts,tsx,json,md,css,scss}\"",
    "format:ci": "prettier --check \"src/**/*.{ts,tsx,json,md,css,scss}\"",
    "quality": "concurrently \"pnpm lint\" \"pnpm format\"",
    "quality:fix": "pnpm lint:fix && pnpm format:fix",
    "quality:report": "pnpm lint:ci && pnpm format:ci"
  },
  "devDependencies": {
    "concurrently": "^8.0.0"
  }
}
```

### 모노레포용 스크립트

```json
{
  "scripts": {
    "lint": "eslint . --ext ts,tsx",
    "lint:packages": "pnpm -r --parallel lint",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "lint:fix:packages": "pnpm -r --parallel lint:fix",
    "format": "prettier --check \"**/*.{ts,tsx,json,md}\"",
    "format:packages": "pnpm -r --parallel format",
    "format:fix": "prettier --write \"**/*.{ts,tsx,json,md}\"",
    "format:fix:packages": "pnpm -r --parallel format:fix",
    "quality": "pnpm lint && pnpm format",
    "quality:packages": "pnpm -r --parallel quality",
    "quality:fix": "pnpm lint:fix && pnpm format:fix",
    "quality:fix:packages": "pnpm -r --parallel quality:fix"
  }
}
```

## Git Hooks 설정

### Husky + lint-staged 설정

```bash
# 의존성 설치
pnpm add -D husky lint-staged

# Husky 초기화
pnpm husky init
```

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": ["eslint --fix", "prettier --write"],
    "*.{json,md,css,scss,html}": ["prettier --write"],
    "*.{ts,tsx,js,jsx}": ["eslint --cache --fix"]
  },
  "scripts": {
    "prepare": "husky"
  }
}
```

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm lint-staged
```

```bash
# .husky/commit-msg
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm commitlint --edit "$1"
```

### 고급 Git Hooks

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# 타입 체크
echo " 타입 체크 중..."
pnpm type-check

# 린트 체크 (변경된 파일만)
echo "🧹 린트 체크 중..."
pnpm lint-staged

# 테스트 실행 (영향받는 테스트만)
echo "🧪 테스트 실행 중..."
pnpm test --passWithNoTests --findRelatedTests $(git diff --cached --name-only --diff-filter=ACM | grep -E '\.(ts|tsx|js|jsx)$' | tr '\n' ' ')

echo " Pre-commit 체크 완료!"
```

## CI/CD 통합

코드 품질 도구들을 CI/CD 파이프라인과 연동하는 상세한 방법은 별도 가이드를 참고하세요:

 **[../cicd/quality-pipeline.md](../cicd/quality-pipeline.md)** - 품질 관리 파이프라인 가이드

### 주요 연동 방법

- **[GitHub Actions](../cicd/github-actions.md)**: GitHub 워크플로우 설정
- **[Azure Pipelines](../cicd/azure-pipelines.md)**: Azure DevOps 파이프라인
- **[품질 자동화](../cicd/automation-tools.md)**: 자동 수정 및 리포팅

## 커스텀 스크립트

### 코드 품질 분석 스크립트

```bash
#!/bin/bash
# scripts/quality-check.sh

set -e

echo " 코드 품질 분석 시작..."

# 타입 체크
echo " TypeScript 타입 체크..."
pnpm type-check

# ESLint 실행
echo " ESLint 검사..."
pnpm lint --format=json --output-file=reports/eslint.json

# Prettier 체크
echo "💅 Prettier 포맷 체크..."
pnpm format

# 복잡도 분석 (선택사항)
if command -v madge &> /dev/null; then
  echo " 의존성 복잡도 분석..."
  madge --circular --warning src/
fi

# 번들 크기 분석 (선택사항)
if [ -f "dist/stats.json" ]; then
  echo "📦 번들 크기 분석..."
  npx bundle-analyzer dist/stats.json
fi

echo " 코드 품질 분석 완료!"
```

### 자동 수정 스크립트

```bash
#!/bin/bash
# scripts/fix-all.sh

set -e

echo " 코드 자동 수정 시작..."

# ESLint 자동 수정
echo " ESLint 자동 수정..."
pnpm lint:fix

# Prettier 자동 포맷팅
echo "💄 Prettier 자동 포맷팅..."
pnpm format:fix

# Import 정렬 (선택사항)
if command -v organize-imports-cli &> /dev/null; then
  echo "📚 Import 정렬..."
  organize-imports-cli src/**/*.ts src/**/*.tsx
fi

# Git add (수정된 파일들)
echo " 변경사항 스테이징..."
git add .

echo " 자동 수정 완료!"
```

### 품질 리포트 생성 스크립트

```javascript
// scripts/quality-report.js
const fs = require("fs");
const path = require("path");
const { execSync } = require("child_process");

async function generateQualityReport() {
  console.log(" 코드 품질 리포트 생성 중...");

  const reports = {};

  try {
    // ESLint 결과
    execSync("pnpm lint --format=json --output-file=reports/eslint.json", {
      stdio: "inherit",
    });
    const eslintReport = JSON.parse(
      fs.readFileSync("reports/eslint.json", "utf8")
    );
    reports.lint = {
      errorCount: eslintReport.reduce((acc, file) => acc + file.errorCount, 0),
      warningCount: eslintReport.reduce(
        (acc, file) => acc + file.warningCount,
        0
      ),
      fileCount: eslintReport.length,
    };
  } catch (error) {
    console.warn("ESLint 실행 실패:", error.message);
    reports.lint = { error: error.message };
  }

  try {
    // Prettier 결과
    const prettierCheck = execSync("pnpm format 2>&1", { encoding: "utf8" });
    reports.format = { status: "passed" };
  } catch (error) {
    reports.format = { status: "failed", output: error.stdout };
  }

  try {
    // TypeScript 컴파일 결과
    const tscOutput = execSync("pnpm type-check 2>&1", { encoding: "utf8" });
    reports.typeCheck = { status: "passed" };
  } catch (error) {
    reports.typeCheck = { status: "failed", output: error.stdout };
  }

  // 리포트 저장
  const reportPath = "reports/quality-summary.json";
  fs.writeFileSync(reportPath, JSON.stringify(reports, null, 2));

  // HTML 리포트 생성
  const htmlReport = `
<!DOCTYPE html>
<html>
<head>
  <title>Code Quality Report</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 40px; }
    .status-passed { color: green; }
    .status-failed { color: red; }
    .summary { background: #f5f5f5; padding: 20px; border-radius: 5px; }
  </style>
</head>
<body>
  <h1>Code Quality Report</h1>
  <div class="summary">
    <h2>Summary</h2>
    <p>ESLint: ${reports.lint.errorCount || 0} errors, ${
    reports.lint.warningCount || 0
  } warnings</p>
    <p>Prettier: <span class="status-${reports.format.status}">${
    reports.format.status
  }</span></p>
    <p>TypeScript: <span class="status-${reports.typeCheck.status}">${
    reports.typeCheck.status
  }</span></p>
  </div>
  <pre>${JSON.stringify(reports, null, 2)}</pre>
</body>
</html>
  `;

  fs.writeFileSync("reports/quality-report.html", htmlReport);

  console.log(" 리포트 생성 완료:");
  console.log("  - JSON: reports/quality-summary.json");
  console.log("  - HTML: reports/quality-report.html");
}

generateQualityReport().catch(console.error);
```

## VS Code 통합

### VS Code 태스크 설정 (.vscode/tasks.json)

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Lint",
      "type": "shell",
      "command": "pnpm",
      "args": ["lint"],
      "group": "build",
      "presentation": {
        "echo": true,
        "reveal": "always",
        "panel": "new"
      },
      "problemMatcher": ["$eslint-stylish"]
    },
    {
      "label": "Lint Fix",
      "type": "shell",
      "command": "pnpm",
      "args": ["lint:fix"],
      "group": "build"
    },
    {
      "label": "Format",
      "type": "shell",
      "command": "pnpm",
      "args": ["format"],
      "group": "build"
    },
    {
      "label": "Format Fix",
      "type": "shell",
      "command": "pnpm",
      "args": ["format:fix"],
      "group": "build"
    },
    {
      "label": "Quality Check",
      "type": "shell",
      "command": "pnpm",
      "args": ["quality"],
      "group": {
        "kind": "build",
        "isDefault": true
      },
      "dependsOrder": "parallel",
      "dependsOn": ["Lint", "Format"]
    }
  ]
}
```

### VS Code 키보드 단축키 (.vscode/keybindings.json)

```json
[
  {
    "key": "ctrl+shift+l",
    "command": "workbench.action.tasks.runTask",
    "args": "Lint Fix"
  },
  {
    "key": "ctrl+shift+p",
    "command": "workbench.action.tasks.runTask",
    "args": "Format Fix"
  },
  {
    "key": "ctrl+shift+q",
    "command": "workbench.action.tasks.runTask",
    "args": "Quality Check"
  }
]
```

## 도커 통합

### Dockerfile에서 품질 검사

```dockerfile
# Dockerfile
FROM node:20-alpine AS quality

WORKDIR /app
COPY package*.json pnpm-lock.yaml ./
RUN npm install -g pnpm && pnpm install --frozen-lockfile

COPY . .

# 코드 품질 검사
RUN pnpm type-check
RUN pnpm lint
RUN pnpm format
RUN pnpm test

# 빌드 단계
FROM quality AS build
RUN pnpm build

# 운영 이미지
FROM node:20-alpine AS production
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/package.json ./
RUN npm install --production
CMD ["node", "dist/index.js"]
```

### Docker Compose 개발 환경

```yaml
# docker-compose.dev.yml
version: "3.8"

services:
  quality-check:
    build:
      context: .
      target: quality
    volumes:
      - .:/app
      - /app/node_modules
    command: pnpm quality:watch

  app:
    build:
      context: .
      target: build
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
    command: pnpm dev
    depends_on:
      - quality-check
```

## 성능 최적화

### 캐시 전략

```bash
# .gitignore에 캐시 파일 추가
.eslintcache
.prettiercache
node_modules/.cache/

# CI에서 캐시 활용
- name: Cache ESLint
  uses: actions/cache@v3
  with:
    path: .eslintcache
    key: eslint-cache-${{ hashFiles('**/eslint.config.mjs') }}-${{ hashFiles('**/pnpm-lock.yaml') }}
```

### 병렬 처리

```json
{
  "scripts": {
    "quality:parallel": "concurrently --kill-others-on-fail \"pnpm lint\" \"pnpm format\" \"pnpm type-check\"",
    "quality:fix:parallel": "concurrently \"pnpm lint:fix\" \"pnpm format:fix\""
  }
}
```

## 문제 해결

### 자주 발생하는 문제

1. **Git Hooks가 실행되지 않음**

   ```bash
   # 실행 권한 확인
   chmod +x .husky/pre-commit

   # Husky 재설치
   pnpm husky install
   ```

2. **CI에서 캐시 문제**

   ```bash
   # 캐시 무효화
   rm -rf .eslintcache .prettiercache
   ```

3. **모노레포에서 설정 충돌**
   ```bash
   # 각 패키지별 설정 확인
   pnpm -r exec eslint --print-config src/index.ts
   ```

### 디버깅 스크립트

```bash
#!/bin/bash
# scripts/debug-quality.sh

echo " 코드 품질 도구 디버깅..."

echo "Node.js 버전:"
node --version

echo "pnpm 버전:"
pnpm --version

echo "ESLint 설정:"
pnpm eslint --print-config src/index.ts

echo "Prettier 설정:"
pnpm prettier --find-config-path src/index.ts

echo "Git hooks 상태:"
ls -la .husky/

echo "캐시 파일:"
ls -la .eslintcache .prettiercache 2>/dev/null || echo "캐시 파일 없음"
```
