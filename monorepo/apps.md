# Apps 디렉토리 가이드

이 문서는 모노레포의 `apps/` 디렉토리에 대한 상세한 가이드입니다.
Apps 디렉토리는 실제 사용자에게 배포되는 애플리케이션들을 포함합니다.

## Apps 디렉토리 개요

`apps/` 디렉토리는 다음과 같은 특징을 가집니다:

- 최종 사용자에게 배포되는 애플리케이션들
- 각각 독립적인 실행 가능한 프로그램
- `packages/`의 공유 라이브러리들을 사용
- 각각 고유한 빌드 및 배포 설정

## 일반적인 Apps 구조

```
apps/
├── web-app/                 # 메인 웹 애플리케이션
│   ├── src/
│   ├── public/
│   ├── package.json
│   ├── vite.config.ts
│   └── tsconfig.json
├── admin-dashboard/         # 관리자 대시보드
│   ├── src/
│   ├── package.json
│   └── next.config.js
├── mobile-app/              # React Native 앱
│   ├── src/
│   ├── package.json
│   └── metro.config.js
└── docs/                    # 문서 사이트
    ├── docs/
    ├── package.json
    └── docusaurus.config.js
```

## 애플리케이션별 package.json 설정

### 웹 애플리케이션 (React + Vite)

```json
{
  "name": "@project/web-app",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite --host --mode dev",
    "build": "pnpm build:prd",
    "build:dev": "tsc -b && vite build --mode dev",
    "build:uat": "tsc -b && vite build --mode uat",
    "build:prd": "tsc -b && vite build --mode prd",
    "build:analyze": "tsc -b && vite build --mode=analyze",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint . --ext ts,tsx --fix",
    "format": "prettier --check \"src/**/*.{ts,tsx,json,md}\"",
    "format:fix": "prettier --write \"src/**/*.{ts,tsx,json,md}\"",
    "typecheck": "tsc --noEmit",
    "preview": "vite preview",
    "clean": "rimraf .turbo dist node_modules"
  },
  "dependencies": {
    "@project/ui-components": "workspace:*",
    "@project/shared-lib": "workspace:*",
    "@project/types": "workspace:*",
    "react": "^19.1.0",
    "react-dom": "^19.1.0",
    "react-router": "^7.6.2"
  },
  "devDependencies": {
    "@types/react": "^19.1.6",
    "@types/react-dom": "^19.1.5",
    "@vitejs/plugin-react": "^4.5.1",
    "typescript": "~5.8.3",
    "vite": "^6.3.5"
  }
}
```

### Next.js 애플리케이션

```json
{
  "name": "@project/admin-dashboard",
  "private": true,
  "version": "0.0.0",
  "scripts": {
    "dev": "next dev --turbo",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "format": "prettier --check \"src/**/*.{ts,tsx,json,md}\"",
    "format:fix": "prettier --write \"src/**/*.{ts,tsx,json,md}\"",
    "typecheck": "tsc --noEmit",
    "clean": "rimraf .turbo .next node_modules"
  },
  "dependencies": {
    "@project/ui-components": "workspace:*",
    "@project/shared-lib": "workspace:*",
    "@project/types": "workspace:*",
    "next": "^15.2.0",
    "react": "^19.1.0",
    "react-dom": "^19.1.0"
  },
  "devDependencies": {
    "@types/react": "^19.1.6",
    "@types/react-dom": "^19.1.5",
    "typescript": "~5.8.3"
  }
}
```

### React Native 애플리케이션

```json
{
  "name": "@project/mobile-app",
  "private": true,
  "version": "0.0.1",
  "scripts": {
    "android": "react-native run-android",
    "ios": "react-native run-ios",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "start": "react-native start",
    "test": "jest",
    "typecheck": "tsc --noEmit",
    "clean": "rimraf .turbo node_modules ios/build android/build"
  },
  "dependencies": {
    "@project/shared-lib": "workspace:*",
    "@project/types": "workspace:*",
    "react": "^19.1.0",
    "react-native": "^0.76.1"
  },
  "devDependencies": {
    "@babel/core": "^7.20.0",
    "@babel/preset-env": "^7.20.0",
    "@babel/runtime": "^7.20.0",
    "@react-native/babel-preset": "^0.76.1",
    "@react-native/eslint-config": "^0.76.1",
    "@react-native/metro-config": "^0.76.1",
    "@react-native/typescript-config": "^0.76.1",
    "@types/react": "^19.1.6",
    "typescript": "~5.8.3"
  }
}
```

## 환경별 빌드 설정

### 환경 변수 관리

각 앱은 환경별로 다른 설정을 가질 수 있습니다:

```
apps/web-app/
├── .env.local           # 로컬 개발용 (git ignore)
├── .env.development     # 개발 환경
├── .env.uat            # UAT 환경
├── .env.production     # 프로덕션 환경
└── .env.example        # 예시 파일
```

### Vite 환경별 설정

```typescript
// apps/web-app/vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig(({ mode }) => ({
  plugins: [react()],
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
  },
  build: {
    outDir: "dist",
    sourcemap: mode !== "prd",
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["react", "react-dom"],
          router: ["react-router"],
        },
      },
    },
  },
  server: {
    port: 3000,
    host: true,
  },
  preview: {
    port: 4173,
    host: true,
  },
}));
```

## 공유 패키지 사용법

### 워크스페이스 패키지 임포트

```typescript
// apps/web-app/src/App.tsx
import { Button, Card } from "@project/ui-components";
import { formatDate, apiClient } from "@project/shared-lib";
import type { User, ApiResponse } from "@project/types";

export default function App() {
  return (
    <Card>
      <Button variant="primary">Hello from {formatDate(new Date())}</Button>
    </Card>
  );
}
```

### 타입 공유

```typescript
// apps/web-app/src/api/users.ts
import { apiClient } from "@project/shared-lib";
import type { User, CreateUserRequest } from "@project/types";

export const getUsers = (): Promise<User[]> => {
  return apiClient.get("/users");
};

export const createUser = (data: CreateUserRequest): Promise<User> => {
  return apiClient.post("/users", data);
};
```

## 개발 서버 설정

### 포트 설정

각 애플리케이션마다 고유한 포트를 사용합니다:

```json
{
  "scripts": {
    "dev:web": "turbo dev --filter=@project/web-app",
    "dev:admin": "turbo dev --filter=@project/admin-dashboard",
    "dev:mobile": "turbo dev --filter=@project/mobile-app"
  }
}
```

### 동시 실행

```bash
# 모든 앱 동시 실행
pnpm dev

# 특정 앱들만 실행
pnpm dev --filter=@project/web-app --filter=@project/admin-dashboard
```

## 배포 설정

### 도커 설정

```dockerfile
# apps/web-app/Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN corepack enable pnpm && pnpm i --frozen-lockfile

# Rebuild the source code only when needed
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN corepack enable pnpm && pnpm build

# Production image
FROM nginx:alpine AS runner
COPY --from=builder /app/dist /usr/share/nginx/html
COPY apps/web-app/nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### CI/CD 워크플로우

```yaml
# .github/workflows/deploy-apps.yml
name: Deploy Apps

on:
  push:
    branches: [main]
    paths: ["apps/**"]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        app: [web-app, admin-dashboard]

    steps:
      - uses: actions/checkout@v4

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build app
        run: pnpm build --filter=@project/${{ matrix.app }}

      - name: Deploy
        run: |
          # 배포 스크립트 실행
          ./scripts/deploy.sh ${{ matrix.app }}
```

## 개발 가이드라인

### 네이밍 컨벤션

- 패키지 이름: `@project/app-name`
- 디렉토리 이름: `kebab-case`
- 컴포넌트 파일: `PascalCase.tsx`
- 유틸리티 파일: `camelCase.ts`

### 의존성 관리

1. **워크스페이스 패키지**: `workspace:*` 프로토콜 사용
2. **외부 패키지**: 가능한 한 루트에서 관리
3. **앱별 특수 의존성**: 해당 앱의 package.json에만 추가

### 코드 공유 원칙

1. **비즈니스 로직**: `packages/shared-lib`으로 분리
2. **UI 컴포넌트**: `packages/ui-components`로 분리
3. **타입 정의**: `packages/types`로 분리
4. **앱별 특수 로직**: 해당 앱 내부에 유지

## 문제 해결

### 의존성 이슈

```bash
# 의존성 재설치
pnpm install --frozen-lockfile

# 특정 앱의 의존성만 재설치
pnpm install --filter=@project/app-name
```

### 빌드 이슈

```bash
# 캐시 클리어 후 빌드
pnpm clean
pnpm build

# 특정 앱만 빌드
pnpm build --filter=@project/app-name
```

### 타입 체크 이슈

```bash
# 전체 타입 체크
pnpm typecheck

# 특정 앱만 타입 체크
pnpm typecheck --filter=@project/app-name
```
