# 모노레포 구성 시 발생하는 문제들과 해결방법

이 문서는 모노레포 구성 과정에서 자주 발생하는 문제들과 검증된 해결방법을 정리합니다.

## 목차

- [구조적 설정 문제](#구조적-설정-문제)
- [의존성 관련 문제](#의존성-관련-문제)
- [TypeScript 설정 문제](#typescript-설정-문제)
- [개발 환경 최적화](#개발-환경-최적화)
- [일반적인 코딩 이슈](#일반적인-코딩-이슈)

## 구조적 설정 문제

### React Router JSX 타입 오류

**문제**: JSX를 사용할 수 없다는 오류 발생

```bash
Cannot use JSX unless the '--jsx' flag is provided.
```

**원인**: TypeScript 설정에서 JSX 관련 설정이 제대로 상속되지 않음

**해결방법**:

1. 공통 TypeScript 설정 확인:

```json
// config/typescript-config/app.json
{
  "extends": "./base.json",
  "compilerOptions": {
    "jsx": "react-jsx", // 이 설정이 중요
    "lib": ["ES2020", "DOM", "DOM.Iterable"]
  }
}
```

2. 앱별 tsconfig.json에서 올바른 확장:

```json
// apps/hooks/tsconfig.json
{
  "extends": "@heart-re-up/typescript-config/app"
}
```

### TypeScript rootDir 설정 문제

**문제**: 라이브러리 빌드 시 rootDir 경로 오류

```bash
error TS6059: File is not under 'rootDir'. 'rootDir' is expected to contain all source files.
```

**원인**: 라이브러리용 TypeScript 설정에서 불필요한 `rootDir` 설정

**해결방법**: 라이브러리 설정에서 rootDir 제거

```json
// config/typescript-config/lib.json
{
  "extends": "./base.json",
  "compilerOptions": {
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    // "rootDir": "./src", // 이 줄 제거
    "composite": true
  }
}
```

### 워크스페이스 패키지 참조 최적화

**문제**: 개발 시 빌드된 패키지를 참조해야 하는 불편함

**해결방법**: TypeScript paths와 Vite alias를 활용한 직접 소스코드 참조

1. **tsconfig.json에 paths 설정**:

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["src/*"],
      "@heart-re-up/hooks": ["../../packages/hooks/src/index.ts"],
      "@heart-re-up/hooks/*": ["../../packages/hooks/src/*"]
    }
  }
}
```

2. **Vite에서 tsconfig paths 자동 읽기**:

```typescript
// vite.config.ts
function getTsconfigPaths() {
  const tsconfig = JSON.parse(readFileSync("tsconfig.json", "utf-8"));
  const paths = tsconfig.compilerOptions?.paths || {};

  const alias: Record<string, string> = {};
  for (const [aliasName, aliasPaths] of Object.entries(paths)) {
    if (Array.isArray(aliasPaths) && aliasPaths.length > 0) {
      const aliasPath = aliasPaths[0].replace(/\/\*$/, "");
      alias[aliasName.replace(/\/\*$/, "")] = resolve(__dirname, aliasPath);
    }
  }
  return alias;
}

export default defineConfig({
  resolve: {
    alias: getTsconfigPaths(),
  },
});
```

**장점**:

- 개발 시: 소스코드 직접 참조로 빠른 Hot Reload
- 빌드 시: package.json exports 사용으로 최적화된 번들

## 의존성 관련 문제

### Radix UI 패키지 선택 오류

**문제**: `@radix-ui/react-card` 패키지를 찾을 수 없음

```bash
ERR_PNPM_FETCH_404  GET https://registry.npmjs.org/@radix-ui%2Freact-card: Not Found - 404
```

**해결방법**: 올바른 패키지 사용

- ❌ `@radix-ui/react-card` (존재하지 않음)
- ✅ `@radix-ui/themes` (Card 컴포넌트 포함)

```json
{
  "dependencies": {
    "@radix-ui/themes": "^3.2.1",
    "@radix-ui/react-accordion": "^1.2.2",
    "@radix-ui/react-separator": "^1.1.1",
    "@radix-ui/react-switch": "^1.1.2",
    "@radix-ui/react-tabs": "^1.1.2"
  }
}
```

### React 19 타입 호환성

**문제**: testing-library와 React 19 버전 불일치 경고

```bash
✕ unmet peer react@^18.0.0: found 19.1.1
```

**해결방법**:

- 현재는 기능상 문제없으므로 경고 무시
- 향후 React 19 지원 버전으로 업데이트 예정

## TypeScript 설정 문제

### 공유 TypeScript 설정의 경로 문제

**문제**: 상대 경로로 인한 include 경로 오류

```bash
error TS18003: No inputs were found in config file
```

**해결방법**: 각 패키지의 tsconfig.json에서 include 경로 명시적 재정의

```json
// packages/hooks/tsconfig.json
{
  "extends": "@heart-re-up/typescript-config/lib",
  "compilerOptions": {
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.tsx", "vite-env.d.ts"],
  "exclude": ["node_modules", "dist", "**/*.test.ts", "**/*.test.tsx"]
}
```

## 개발 환경 최적화

### DRY 원칙 적용 - 경로 설정 통합

**문제**: tsconfig.json과 vite.config.ts에서 경로 설정 중복 관리

**해결방법**: tsconfig paths를 단일 소스로 사용

```typescript
// vite.config.ts에서 tsconfig.json 읽어서 자동 설정
function getTsconfigPaths() {
  // tsconfig.json의 paths 설정을 읽어서 Vite alias로 변환
}
```

### 모노레포 개발 경험 향상

**전략**:

- 개발 시: 직접 소스코드 참조 (빠른 HMR)
- 빌드 시: exports 필드 사용 (최적화된 번들)
- 타입 체킹: 실시간 소스코드 기반

## 일반적인 코딩 이슈

다음은 개발 과정에서 자연스럽게 발생하는 일반적인 이슈들입니다:

### 사용하지 않는 import 정리

```typescript
// ❌ 사용하지 않는 import
import { useState, useEffect } from "react"; // useEffect 미사용
import { expect, afterEach } from "vitest"; // expect 미사용

// ✅ 필요한 것만 import
import { useState } from "react";
import { afterEach } from "vitest";
```

### 훅 사용 시 구조분해 할당 순서

```typescript
// useToggle 훅 사용 시
const [value, toggle, setValue] = useToggle(false);
```

### 타입 명시

```typescript
// 콜백에서 타입 명시
setSettings((prev: typeof settings) => ({ ...prev, theme: value }));
```

## 모범 사례

### 단계적 문제 해결

1. **타입 검사 먼저**: `pnpm typecheck`로 TypeScript 오류 우선 해결
2. **빌드 테스트**: `pnpm build`로 빌드 오류 확인
3. **개발 서버 실행**: `pnpm dev`로 런타임 오류 확인

### 설정 파일 검증

```bash
# 모든 패키지 타입 검사
pnpm typecheck

# 모든 패키지 린트 검사
pnpm lint

# 모든 패키지 포맷 검사
pnpm format
```

### 디버깅 팁

####rbo 캐시 문제 시

```bash
# 캐시 정리
pnpm clean:cache

# 전체 정리 후 재설치
pnpm clean && pnpm install
```

####peScript 경로 매핑 문제 시

```bash
# TypeScript 컴파일러 추적
tsc --traceResolution --noEmit
```

이러한 문제들과 해결방법을 미리 알고 있으면 모노레포 구성 시간을 크게 단축할 수 있습니다.
