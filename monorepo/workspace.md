# 워크스페이스 설정 가이드

패키지 매니저, 워크스페이스, 빌드 시스템 설정을 다룹니다.

## 1. pnpm Workspace 설정

### pnpm-workspace.yaml

```yaml
packages:
  - apps/*
  - packages/*
```

### .pnpmrc 설정

```ini
# 패키지 설치 설정
auto-install-peers=true
shamefully-hoist=true
strict-peer-dependencies=false

# 패키지 레지스트리
registry=https://registry.npmjs.org/

# 사설 레지스트리 (필요시)
@company:registry=https://npm.company.com

# 성능 최적화
network-concurrency=16
fetch-retries=3
fetch-retry-mintimeout=10000
fetch-retry-maxtimeout=60000

# pnpm 전용 설정
prefer-workspace-packages=true
link-workspace-packages=deep
shared-workspace-lockfile=true

# 보안
audit-level=moderate
```

### 특정 패키지용 설정

```ini
# React 19 타입 관련 이슈 해결 (권장)
shamefully-hoist=false
public-hoist-pattern[]=*eslint*
public-hoist-pattern[]=*prettier*
public-hoist-pattern[]=*@types*

#  주의: React 19 사용 시 testing-library 호환성 경고 발생 가능
# 현재는 기능상 문제없으므로 무시 가능
```

## 2. 루트 package.json 설정

```json
{
  "name": "@project/root",
  "version": "1.0.0",
  "private": true,
  "packageManager": "pnpm@10.10.0",
  "scripts": {
    "build": "turbo build",
    "dev": "turbo dev",
    "lint": "turbo lint",
    "lint:fix": "turbo lint:fix",
    "format": "turbo format",
    "format:fix": "turbo format:fix",
    "typecheck": "turbo typecheck",
    "clean": "turbo clean && rm -rf .turbo node_modules",
    "clean:cache": "turbo clean",
    "preview": "turbo preview"
  },
  "devDependencies": {
    "@eslint/js": "^9.28.0",
    "eslint": "^9.28.0",
    "eslint-config-prettier": "^9.1.0",
    "eslint-plugin-only-warn": "^1.1.0",
    "eslint-plugin-react": "^7.37.5",
    "eslint-plugin-react-hooks": "^5.2.0",
    "eslint-plugin-react-refresh": "^0.4.20",
    "eslint-plugin-turbo": "^2.5.4",
    "globals": "^15.15.0",
    "prettier": "^3.5.3",
    "turbo": "^2.5.4",
    "typescript": "~5.8.3",
    "typescript-eslint": "^8.33.1",
    "rimraf": "^6.0.1"
  },
  "engines": {
    "node": ">=18",
    "pnpm": ">=8"
  }
}
```

## 3. Turbo 설정

### turbo.json

```json
{
  "$schema": "https://turbo.build/schema.json",
  "ui": "tui",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [
        "dist/**", 
        ".next/**", 
        "!.next/cache/**", 
        "build/**",
        "apps/backend/dist/**"
      ]
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "lint:fix": {
      "dependsOn": ["^lint:fix"]
    },
    "format": {
      "dependsOn": ["^format"]
    },
    "format:fix": {
      "dependsOn": ["^format:fix"]
    },
    "typecheck": {
      "dependsOn": ["^typecheck"]
    },
    "test": {
      "dependsOn": ["^test"]
    },
    "test:watch": {
      "cache": false,
      "persistent": true
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "preview": {
      "cache": false,
      "persistent": true
    },
    "clean": {
      "cache": false
    }
  },
  "globalDependencies": [
    "**/.env.*local",
    ".env",
    ".env.local",
    ".env.development",
    ".env.production"
  ]
}
```

## 4. 모듈별 package.json 패턴

### 애플리케이션 모듈 (apps/main-app/package.json)

```json
{
  "name": "@project/main-app",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite --host --mode dev",
    "build": "pnpm build:prd",
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
    "@project/shared-lib": "workspace:*",
    "@project/ui-components": "workspace:*",
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

### 라이브러리 모듈 (packages/shared-lib/package.json)

```json
{
  "name": "@project/shared-lib",
  "version": "1.0.0",
  "description": "Shared Library",
  "private": false,
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./hooks": {
      "types": "./dist/hooks.d.ts",
      "import": "./dist/hooks.js",
      "require": "./dist/hooks.cjs"
    }
  },
  "files": ["dist", "README.md"],
  "sideEffects": false,
  "scripts": {
    "build": "vite build --config vite.lib.config.ts",
    "build:watch": "vite build --config vite.lib.config.ts --watch",
    "test": "vitest",
    "test:coverage": "vitest run --coverage",
    "prebuild": "rimraf dist",
    "prepublishOnly": "pnpm test && pnpm build"
  },
  "peerDependencies": {
    "react": ">=18",
    "react-dom": ">=18"
  },
  "devDependencies": {
    "@vitejs/plugin-react": "^4.3.1",
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.0.0",
    "@vitest/coverage-v8": "^1.0.0",
    "jsdom": "^23.0.0",
    "vite": "^5.3.1",
    "vite-plugin-dts": "^3.9.1",
    "vitest": "^1.0.0"
  }
}
```

## 모듈 간 연동 방법

### Workspace Protocol 사용

```json
{
  "dependencies": {
    "@project/shared-lib": "workspace:*",
    "@project/ui-components": "workspace:*"
  }
}
```

## 의존성 관리

```bash
# 특정 모듈에 패키지 추가
pnpm add axios --filter=@project/main-app

# 워크스페이스 패키지 추가
pnpm add @project/shared-lib --workspace --filter=@project/main-app
```

## NPM 배포 설정 (선택사항)

```json
{
  "publishConfig": {
    "registry": "https://registry.npmjs.org/"
  }
}
```

로컬 개발 시 pnpm link 사용:

```bash
# 패키지 디렉토리에서
pnpm link --global

# 사용할 프로젝트에서
pnpm link --global @project/shared-lib
```
