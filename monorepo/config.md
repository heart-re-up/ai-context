# 모노레포 설정 관리 전략

모노레포에서 ESLint, TypeScript, Prettier 등의 설정 파일을 체계적으로 구조화하고 모듈간 공유하는 방법에 대한 가이드입니다.

>  **상세한 도구별 설정 방법**은 [`quality/` 디렉토리](../quality/README.md)를 참고하세요.

## 설정 파일 구조 전략

###  별도 config 디렉토리 (권장)

```
project-root/
├── config/                    # 모든 설정 파일 중앙 관리
│   ├── eslint-config/         # ESLint 설정 패키지
│   ├── typescript-config/     # TypeScript 설정 패키지
│   ├── prettier-config/       # Prettier 설정 패키지
│   ├── vite-config/          # Vite 설정 패키지
│   └── vitest-config/        # Vitest 설정 패키지
├── packages/                  # 비즈니스 로직 패키지들
│   ├── ui-components/
│   ├── shared-lib/
│   └── types/
└── apps/                     # 애플리케이션들
    ├── web/
    └── admin/
```

###  장점

- **중앙화**: 모든 설정이 한 곳에서 관리됨
- **재사용성**: 다양한 모듈 타입별 설정 제공
- **유지보수**: 설정 변경 시 한 번만 수정
- **일관성**: 프로젝트 전체에 동일한 규칙 적용
- **확장성**: 새로운 도구 설정 추가 용이
- **명확성**: 설정과 비즈니스 로직 분리

## 설정 패키지 구조

### ESLint 설정 패키지

```
config/eslint-config/
├── package.json         # 패키지 정의
├── base.mjs            # 기본 ESLint 설정 ( globals import 필수)
├── react.mjs           # React 프로젝트용 (React 19 호환)
├── node.mjs            # Node.js 프로젝트용 ( globals import 필수)
└── lib.mjs             # 라이브러리용 (더 엄격)
```

**패키지 설정 예시:**

```json
{
  "name": "@project/eslint-config",
  "exports": {
    ".": "./base.mjs",
    "./react": "./react.mjs",
    "./node": "./node.mjs",
    "./lib": "./lib.mjs"
  },
  "files": ["*.mjs", "*.js"]
}
```

> ** `files` 옵션 설명**:
>
> - **NPM 배포용**: NPM에 패키지를 퍼블리시할 때 포함할 파일들을 지정
> - **모노레포에서는 무관**: 워크스페이스 내에서는 `files` 와 관계없이 모든 파일에 접근 가능
> - **개발 시**: `import from "@project/eslint-config/react"`는 실제 파일 경로로 직접 참조됨

>  **ESLint 상세 설정 방법**: [quality/eslint.md](../quality/eslint.md)
> 
>  **주의사항**: 
> - `base.mjs`와 `node.mjs`에서 `globals` import 누락 시 오류 발생
> - React 19 사용 시 `react/jsx-uses-react` 규칙 비활성화 필요

### TypeScript 설정 패키지

```
config/typescript-config/
├── package.json         # 패키지 정의
├── base.json           # 기본 TypeScript 설정
├── app.json            # 애플리케이션용 (DOM, React)
├── lib.json            # 라이브러리용 (declaration 생성)
└── node.json           # Node.js 서버용
```

**패키지 설정 예시:**

```json
{
  "name": "@project/typescript-config",
  "exports": {
    "./base": "./base.json",
    "./app": "./app.json",
    "./lib": "./lib.json",
    "./node": "./node.json"
  }
}
```

>  **TypeScript 상세 설정 방법**: [typescript.md](../typescript.md)

### Prettier 설정 패키지

```
config/prettier-config/
├── package.json         # 패키지 정의
└── index.js            # Prettier 설정
```

**패키지 설정 예시:**

```json
{
  "name": "@project/prettier-config",
  "main": "index.js"
}
```

>  **Prettier 상세 설정 방법**: [quality/prettier.md](../quality/prettier.md)

## 워크스페이스 통합

### 1. pnpm-workspace.yaml 업데이트

```yaml
packages:
  - apps/*
  - packages/*
  - config/* # 설정 패키지 추가
```

### 2. 설정 패키지 vs NPM 패키지

#### 워크스페이스 내 설정 패키지 (권장)

```json
// config/eslint-config/package.json
{
  "name": "@project/eslint-config",
  "private": true, // 외부 배포 안함
  "version": "1.0.0",
  "exports": {
    ".": "./base.mjs",
    "./react": "./react.mjs"
  }
  // files 옵션 불필요 - 워크스페이스 내에서는 모든 파일 접근 가능
}
```

#### 외부 배포용 설정 패키지

```json
// config/eslint-config/package.json
{
  "name": "@company/eslint-config",
  "private": false, // NPM 배포
  "version": "1.0.0",
  "files": ["*.mjs", "*.js", "README.md"], // 배포에 포함할 파일들
  "exports": {
    ".": "./base.mjs",
    "./react": "./react.mjs"
  }
}
```

### 3. 의존성 설치 방법

#### 설정 패키지를 루트에서 관리 (권장)

```json
// 루트 package.json
{
  "devDependencies": {
    "@project/eslint-config": "workspace:*",
    "@project/typescript-config": "workspace:*",
    "@project/prettier-config": "workspace:*",

    // 실제 도구들도 루트에서 관리
    "eslint": "^9.0.0",
    "prettier": "^3.0.0",
    "typescript": "^5.0.0"
  }
}
```

#### 각 모듈에서 개별 설치

```json
// apps/web/package.json
{
  "devDependencies": {
    "@project/eslint-config": "workspace:*", // 워크스페이스 참조
    "eslint": "^9.0.0" // 실제 도구
  }
}
```

## 모듈간 참조 방법

### 파일 참조의 이해

#### 워크스페이스에서의 파일 접근

```bash
# 모노레포 구조
project-root/
├── config/eslint-config/
│   ├── package.json          # "name": "@project/eslint-config"
│   ├── base.mjs             # 실제 파일
│   └── react.mjs            # 실제 파일
└── apps/web/
    └── eslint.config.mjs     # 설정을 import하는 파일
```

```javascript
// apps/web/eslint.config.mjs
// 이 import는 다음과 같이 해석됩니다:
import reactConfig from "@project/eslint-config/react";
//                     ↓
//         실제로는: ../../config/eslint-config/react.mjs

export default reactConfig;
```

> ** 핵심**: 워크스페이스에서 `@project/eslint-config/react`는
> `config/eslint-config/react.mjs` 파일을 직접 참조합니다.
> `files` 옵션과는 **무관**합니다.

### 애플리케이션에서 사용

```javascript
// apps/web/eslint.config.mjs
import reactConfig from "@project/eslint-config/react";

export default reactConfig;
```

```json
// apps/web/tsconfig.json
{
  "extends": "@project/typescript-config/app",
  "compilerOptions": {
    "paths": {
      "@/*": ["src/*"],
      "@project/ui-components": ["../../packages/ui-components/src"]
    }
  }
}
```

```json
// apps/web/package.json
{
  "prettier": "@project/prettier-config"
}
```

### 패키지에서 사용

```javascript
// packages/ui-components/eslint.config.mjs
import libConfig from "@project/eslint-config/lib";
import reactConfig from "@project/eslint-config/react";

export default [...libConfig, ...reactConfig];
```

```json
// packages/ui-components/tsconfig.json
{
  "extends": "@project/typescript-config/lib",
  "compilerOptions": {
    "jsx": "react-jsx"
  }
}
```

### 설정 확장 및 재정의

```javascript
// apps/admin/eslint.config.mjs - 설정 확장
import reactConfig from "@project/eslint-config/react";

export default [
  ...reactConfig,
  {
    // 관리자 앱 전용 규칙 추가
    rules: {
      "no-console": "off", // console 허용
    },
  },
];
```

```javascript
// packages/server/eslint.config.mjs - 다른 설정 사용
import nodeConfig from "@project/eslint-config/node";

export default nodeConfig;
```

## Turbo.json 업데이트

```json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**"]
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "format": {
      "dependsOn": ["^format"]
    }
  }
}
```

## 설정 변경 시 워크플로우

1. **설정 수정**: `config/` 디렉토리에서 해당 설정 파일 수정
2. **버전 업데이트**: 설정 패키지의 버전 업데이트 (선택사항)
3. **의존성 업데이트**: 필요시 사용하는 앱/패키지에서 설정 패키지 버전 업데이트
4. **테스트**: 모든 앱과 패키지에서 설정이 올바르게 적용되는지 확인

```bash
# 모든 패키지에서 설정 테스트
pnpm -r lint
pnpm -r format
pnpm -r typecheck
```

## 마이그레이션 가이드

### 기존 프로젝트에서 적용하기

1. **config 디렉토리 생성**

   ```bash
   mkdir -p config/{eslint-config,typescript-config,prettier-config}
   ```

2. **설정 파일 이동 및 패키지화**

   - 기존 설정 파일들을 config 디렉토리로 이동
   - 각 설정을 독립 패키지로 구성

3. **워크스페이스 설정 업데이트**

   - `pnpm-workspace.yaml`에 config 추가
   - 루트 `package.json`에 설정 패키지 의존성 추가

4. **기존 프로젝트 설정 업데이트**
   - 각 앱과 패키지의 설정 파일을 새로운 설정 패키지를 사용하도록 수정

### 마이그레이션 스크립트 예시

```bash
#!/bin/bash
# scripts/migrate-config.sh

echo " 설정 파일 마이그레이션 시작..."

# 1. config 디렉토리 생성
mkdir -p config/{eslint-config,typescript-config,prettier-config}

# 2. 기존 설정 파일 이동
if [ -f "eslint.config.mjs" ]; then
  mv eslint.config.mjs config/eslint-config/base.mjs
fi

if [ -f "tsconfig.json" ]; then
  cp tsconfig.json config/typescript-config/base.json
fi

if [ -f "prettier.config.mjs" ]; then
  mv prettier.config.mjs config/prettier-config/index.js
fi

# 3. 패키지 파일 생성
echo '{"name": "@project/eslint-config", "version": "1.0.0"}' > config/eslint-config/package.json
echo '{"name": "@project/typescript-config", "version": "1.0.0"}' > config/typescript-config/package.json
echo '{"name": "@project/prettier-config", "version": "1.0.0"}' > config/prettier-config/package.json

echo " 마이그레이션 완료!"
```

## 모범 사례

1. **설정 패키지 네이밍**: `@project/` 접두사 사용
2. **버전 관리**: 설정 변경 시 의미있는 버전 업데이트
3. **문서화**: 각 설정 패키지에 README.md 추가
4. **테스트**: 설정 변경 후 모든 모듈에서 동작 확인
5. **점진적 적용**: 한 번에 모든 설정을 변경하지 말고 단계적으로 적용

이 구조를 통해 모노레포의 모든 설정을 체계적이고 효율적으로 관리할 수 있습니다.
