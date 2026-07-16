# Node.js 스크립트 관리 가이드

Node.js 프로젝트에서 자주 사용되는 npm scripts 패턴과 환경별 실행 방법에 대한 가이드입니다.

## 스크립트 분류

### 개발 스크립트

- **dev**: 개발 서버 시작
- **start**: 프로덕션 서버 시작
- **build**: 프로덕션 빌드

### 코드 품질 스크립트

- **lint**: 코드 린팅
- **format**: 코드 포맷팅
- **typecheck**: 타입 체크

### 테스트 스크립트

- **test**: 테스트 실행
- **test:watch**: 감시 모드 테스트
- **test:coverage**: 커버리지 포함 테스트

### 유틸리티 스크립트

- **clean**: 임시 파일 정리
- **prebuild**: 빌드 전 정리
- **postinstall**: 설치 후 작업

## 기본 스크립트 구성

### 일반적인 Node.js 프로젝트

```json
{
  "scripts": {
    "dev": "nodemon src/index.ts",
    "start": "NODE_ENV=production node dist/index.js",
    "build": "npm run build:clean && npm run build:compile",
    "build:clean": "rimraf dist",
    "build:compile": "tsc",
    "build:watch": "tsc --watch",

    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:ci": "jest --ci --coverage --watchAll=false",

    "lint": "eslint src/**/*.{js,ts}",
    "lint:fix": "eslint src/**/*.{js,ts} --fix",
    "format": "prettier --check \"src/**/*.{js,ts,json,md}\"",
    "format:fix": "prettier --write \"src/**/*.{js,ts,json,md}\"",
    "typecheck": "tsc --noEmit",

    "clean": "rimraf dist node_modules/.cache",
    "prebuild": "npm run clean",
    "postinstall": "husky install"
  }
}
```

### Express.js 서버

```json
{
  "scripts": {
    "dev": "nodemon --exec ts-node src/server.ts",
    "start": "NODE_ENV=production node dist/server.js",
    "build": "tsc && npm run copy:assets",
    "copy:assets": "cp -r src/public dist/",

    "migrate": "knex migrate:latest",
    "migrate:rollback": "knex migrate:rollback",
    "seed": "knex seed:run",

    "docker:build": "docker build -t my-app .",
    "docker:run": "docker run -p 3000:3000 my-app"
  }
}
```

### 모노레포 루트

```json
{
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "test": "turbo test",
    "lint": "turbo lint",
    "format": "turbo format",
    "typecheck": "turbo typecheck",
    "clean": "turbo clean && rm -rf .turbo node_modules"
  }
}
```

## 환경별 스크립트 실행

### 환경 변수를 사용한 분기

```json
{
  "scripts": {
    "dev": "NODE_ENV=development nodemon src/server.ts",
    "dev:staging": "NODE_ENV=staging nodemon src/server.ts",
    "start:prod": "NODE_ENV=production node dist/server.js",
    "start:staging": "NODE_ENV=staging node dist/server.js"
  }
}
```

### cross-env를 사용한 크로스 플랫폼 지원

```json
{
  "scripts": {
    "dev": "cross-env NODE_ENV=development nodemon src/server.ts",
    "build:prod": "cross-env NODE_ENV=production tsc",
    "test": "cross-env NODE_ENV=test jest"
  },
  "devDependencies": {
    "cross-env": "^7.0.3"
  }
}
```

### dotenv를 사용한 환경 파일 로드

```json
{
  "scripts": {
    "dev": "dotenv -e .env.development -- nodemon src/server.ts",
    "dev:staging": "dotenv -e .env.staging -- nodemon src/server.ts",
    "start:prod": "dotenv -e .env.production -- node dist/server.js"
  },
  "devDependencies": {
    "dotenv-cli": "^7.0.0"
  }
}
```

## 고급 스크립트 패턴

### 병렬 실행

```json
{
  "scripts": {
    "dev:all": "concurrently \"npm:dev:client\" \"npm:dev:server\"",
    "dev:client": "cd client && npm run dev",
    "dev:server": "cd server && npm run dev",
    "build:all": "npm run build:client && npm run build:server",
    "test:all": "npm run test:unit && npm run test:integration"
  },
  "devDependencies": {
    "concurrently": "^8.0.0"
  }
}
```

### 조건부 실행

```json
{
  "scripts": {
    "build": "npm run build:clean && npm run build:compile && npm run build:assets",
    "build:assets": "test -d src/assets && cp -r src/assets dist/ || echo 'No assets to copy'",
    "test": "npm run test:lint && npm run test:unit",
    "test:ci": "npm run test && npm run test:coverage"
  }
}
```

### 매개변수 전달

```json
{
  "scripts": {
    "test": "jest",
    "test:file": "jest --testNamePattern",
    "migrate": "knex migrate:latest",
    "migrate:make": "knex migrate:make",
    "docker:run": "docker run -p 3000:3000"
  }
}
```

**사용 예시:**

```bash
# 특정 테스트 파일 실행
npm run test:file -- user.test.ts

# 새 마이그레이션 파일 생성
npm run migrate:make -- add_users_table

# 다른 포트로 Docker 실행
npm run docker:run -- -p 3001:3000
```

## 개발 도구 통합

### Nodemon 설정

```json
// nodemon.json
{
  "watch": ["src"],
  "ext": "ts,js,json",
  "ignore": ["src/**/*.test.ts"],
  "exec": "ts-node src/index.ts",
  "env": {
    "NODE_ENV": "development"
  }
}
```

```json
{
  "scripts": {
    "dev": "nodemon",
    "dev:debug": "nodemon --inspect src/index.ts"
  }
}
```

### PM2 프로세스 관리

```json
{
  "scripts": {
    "start": "pm2 start ecosystem.config.js",
    "stop": "pm2 stop ecosystem.config.js",
    "restart": "pm2 restart ecosystem.config.js",
    "logs": "pm2 logs",
    "monitor": "pm2 monit"
  }
}
```

```javascript
// ecosystem.config.js
module.exports = {
  apps: [
    {
      name: "my-app",
      script: "dist/index.js",
      instances: "max",
      exec_mode: "cluster",
      env: {
        NODE_ENV: "development",
      },
      env_production: {
        NODE_ENV: "production",
      },
    },
  ],
};
```

## CI/CD 최적화 스크립트

### GitHub Actions 최적화

```json
{
  "scripts": {
    "ci:install": "npm ci --frozen-lockfile",
    "ci:build": "npm run build",
    "ci:test": "npm run test:ci",
    "ci:lint": "npm run lint",
    "ci:typecheck": "npm run typecheck",
    "ci:all": "npm run ci:install && npm run ci:lint && npm run ci:typecheck && npm run ci:test && npm run ci:build"
  }
}
```

### Docker 빌드 최적화

```json
{
  "scripts": {
    "docker:build": "docker build -t my-app:latest .",
    "docker:build:dev": "docker build -f Dockerfile.dev -t my-app:dev .",
    "docker:push": "docker push my-app:latest",
    "docker:run:dev": "docker run --env-file .env.development -p 3000:3000 my-app:dev"
  }
}
```

## 성능 최적화

### 빌드 성능

```json
{
  "scripts": {
    "build": "npm run build:clean && npm run build:compile",
    "build:watch": "tsc --watch --preserveWatchOutput",
    "build:analyze": "npm run build && bundlesize",
    "prebuild": "rimraf dist",
    "postbuild": "npm run build:copy-assets"
  }
}
```

### 테스트 성능

```json
{
  "scripts": {
    "test": "jest --passWithNoTests",
    "test:changed": "jest --onlyChanged",
    "test:watch": "jest --watch --onlyChanged",
    "test:debug": "node --inspect-brk node_modules/.bin/jest --runInBand"
  }
}
```

## 스크립트 조합 패턴

### Pre/Post 훅 활용

```json
{
  "scripts": {
    "build": "webpack",
    "prebuild": "npm run clean && npm run lint",
    "postbuild": "npm run test",

    "install": "npm ci",
    "postinstall": "npm run build:deps",

    "test": "jest",
    "pretest": "npm run lint",
    "posttest": "npm run coverage:report"
  }
}
```

### 환경별 스크립트 매트릭스

```json
{
  "scripts": {
    "dev": "npm run dev:development",
    "dev:development": "cross-env NODE_ENV=development nodemon",
    "dev:staging": "cross-env NODE_ENV=staging nodemon",
    "dev:production": "cross-env NODE_ENV=production node dist/index.js",

    "build": "npm run build:production",
    "build:development": "cross-env NODE_ENV=development tsc",
    "build:staging": "cross-env NODE_ENV=staging tsc",
    "build:production": "cross-env NODE_ENV=production tsc",

    "test": "npm run test:development",
    "test:development": "cross-env NODE_ENV=test jest",
    "test:ci": "cross-env NODE_ENV=test CI=true jest --ci"
  }
}
```

## 문제 해결

### 권한 문제

```json
{
  "scripts": {
    "clean": "rimraf dist || rm -rf dist",
    "build": "npm run clean && tsc",
    "postinstall": "test -f .husky/pre-commit && chmod +x .husky/pre-commit || echo 'Husky not installed'"
  }
}
```

### 메모리 부족

```json
{
  "scripts": {
    "build": "node --max-old-space-size=4096 node_modules/typescript/bin/tsc",
    "test": "node --max-old-space-size=4096 node_modules/.bin/jest"
  }
}
```

### 플랫폼별 차이

```json
{
  "scripts": {
    "clean": "rimraf dist",
    "clean:win": "if exist dist rmdir /s /q dist",
    "clean:unix": "rm -rf dist",
    "copy:assets": "cpy src/assets/* dist/assets/"
  },
  "devDependencies": {
    "cpy-cli": "^4.0.0",
    "rimraf": "^5.0.0"
  }
}
```

## 모범 사례

1. **명명 규칙**: 일관된 스크립트 명명 (dev, build, test, lint 등)
2. **단일 책임**: 각 스크립트는 하나의 명확한 역할
3. **조합 가능**: 작은 스크립트들을 조합해서 복잡한 작업 구성
4. **환경 독립적**: 플랫폼에 관계없이 동작
5. **오류 처리**: 실패 시 적절한 종료 코드 반환
6. **문서화**: 복잡한 스크립트는 주석으로 설명

## 관련 문서

- [environment-variables.md](./environment-variables.md) - 환경 변수 설정
- [version-management.md](./version-management.md) - Node.js 버전 관리
- [monorepo/workspace.md](../monorepo/workspace.md) - 모노레포 스크립트 관리
