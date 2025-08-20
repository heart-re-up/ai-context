# 모노레포 빠른 시작 가이드

이 가이드는 실제 프로젝트에서 검증된 모노레포 구성을 빠르게 시작할 수 있도록 도와줍니다.

##  5분 만에 모노레포 구성하기

### 1단계: 기본 구조 생성

```bash
# 프로젝트 디렉토리 생성
mkdir my-monorepo && cd my-monorepo

# 기본 디렉토리 구조 생성
mkdir -p apps packages config
```

### 2단계: 워크스페이스 설정

**pnpm-workspace.yaml**
```yaml
packages:
  - apps/*
  - packages/*
  - config/*
```

**package.json**
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
    "typecheck": "turbo typecheck"
  },
  "devDependencies": {
    "turbo": "^2.5.4",
    "typescript": "~5.8.3",
    "prettier": "^3.5.3"
  }
}
```

**turbo.json**
```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", "build/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "typecheck": {
      "dependsOn": ["^typecheck"]
    }
  }
}
```

### 3단계: 공통 설정 생성

#### TypeScript 설정
```bash
mkdir -p config/typescript-config
```

**config/typescript-config/package.json**
```json
{
  "name": "@project/typescript-config",
  "version": "1.0.0",
  "private": true,
  "exports": {
    "./base": "./base.json",
    "./app": "./app.json",
    "./lib": "./lib.json"
  }
}
```

**config/typescript-config/base.json**
```json
{
  "compilerOptions": {
    "target": "ES2016",
    "lib": ["ES2020"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "skipLibCheck": true,
    "isolatedModules": true,
    "noEmit": true
  }
}
```

### 4단계: React 앱 생성

```bash
mkdir -p apps/frontend
```

**apps/frontend/package.json**
```json
{
  "name": "@project/frontend",
  "private": true,
  "scripts": {
    "dev": "vite --host",
    "build": "tsc -b && vite build",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "react": "^19.1.0",
    "react-dom": "^19.1.0"
  },
  "devDependencies": {
    "@project/typescript-config": "workspace:*",
    "@types/react": "^19.1.6",
    "@types/react-dom": "^19.1.5",
    "@vitejs/plugin-react": "^4.5.1",
    "vite": "^6.3.5"
  }
}
```

### 5단계: 의존성 설치 및 실행

```bash
# 의존성 설치
pnpm install

# 타입 검사
pnpm typecheck

# 개발 서버 실행
pnpm dev
```

##  체크리스트

### 필수 파일들
- [ ] `pnpm-workspace.yaml`
- [ ] `turbo.json`
- [ ] 루트 `package.json`
- [ ] 공통 TypeScript 설정
- [ ] `.gitignore`

### 검증 단계
- [ ] `pnpm install` 성공
- [ ] `pnpm typecheck` 통과
- [ ] `pnpm build` 성공
- [ ] `pnpm dev` 실행 확인

##  추가 패키지 생성

### UI 컴포넌트 라이브러리

```bash
mkdir -p packages/ui-components/src
```

**packages/ui-components/package.json**
```json
{
  "name": "@project/ui-components",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "scripts": {
    "build": "vite build",
    "typecheck": "tsc --noEmit"
  },
  "peerDependencies": {
    "react": ">=19"
  },
  "devDependencies": {
    "@project/typescript-config": "workspace:*",
    "react": "^19.1.0",
    "vite": "^6.3.5"
  }
}
```

### NestJS 백엔드

```bash
mkdir -p apps/backend/src
```

**apps/backend/package.json**
```json
{
  "name": "@project/backend",
  "private": true,
  "scripts": {
    "dev": "nest start --watch",
    "build": "nest build",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "@nestjs/common": "^10.0.0",
    "@nestjs/core": "^10.0.0",
    "@nestjs/platform-express": "^10.0.0"
  },
  "devDependencies": {
    "@project/typescript-config": "workspace:*",
    "@nestjs/cli": "^10.0.0"
  }
}
```

##  자주 사용하는 명령어

```bash
# 특정 앱만 실행
pnpm dev --filter=@project/frontend

# 특정 앱에 패키지 추가
pnpm add axios --filter=@project/frontend

# 워크스페이스 패키지 참조
pnpm add @project/ui-components --filter=@project/frontend

# 모든 패키지 빌드
pnpm build

# 캐시 정리
pnpm clean
```

##  주의사항

1. **React 19 사용 시**: `React.JSX.Element` 타입 사용
2. **Tailwind CSS**: 3.x 버전 사용 권장 (4.x는 설정 복잡)
3. **타입 의존성**: 각 패키지에 필요한 `@types/*` 명시적 설치
4. **ESLint 설정**: `globals` import 누락 주의

이 가이드를 따르면 5분 내에 작동하는 모노레포를 구성할 수 있습니다!