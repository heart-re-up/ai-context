# 모노레포 워크스페이스 최적화 가이드

모노레포에서 개발 경험을 최적화하기 위한 고급 설정 방법입니다.

## 개발 시 직접 소스코드 참조

### 문제점

- 기본적으로 워크스페이스 패키지는 빌드된 결과물을 참조
- 개발 시마다 패키지를 빌드해야 하는 불편함
- Hot Module Replacement(HMR)가 제대로 작동하지 않음

### 해결책: TypeScript Paths + Vite Alias

#### 1. TypeScript 경로 매핑 설정

```json
// apps/hooks/tsconfig.json
{
  "extends": "@heart-re-up/typescript-config/app",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@heart-re-up/hooks": ["../../packages/hooks/src/index.ts"],
      "@heart-re-up/hooks/*": ["../../packages/hooks/src/*"]
    }
  }
}
```

#### 2. Vite에서 tsconfig paths 자동 읽기

```typescript
// apps/hooks/vite.config.ts
import { defineConfig } from "vite";
import { resolve } from "path";
import { readFileSync } from "fs";

// tsconfig.json에서 paths를 읽어서 Vite alias로 변환하는 함수
function getTsconfigPaths() {
  try {
    const tsconfigPath = resolve(__dirname, "tsconfig.json");
    const tsconfig = JSON.parse(readFileSync(tsconfigPath, "utf-8"));
    const paths = tsconfig.compilerOptions?.paths || {};

    const alias: Record<string, string> = {};

    for (const [aliasName, aliasPaths] of Object.entries(paths)) {
      if (Array.isArray(aliasPaths) && aliasPaths.length > 0) {
        // tsconfig의 baseUrl 기준 경로를 절대 경로로 변환
        const aliasPath = aliasPaths[0].replace(/\/\*$/, ""); // /* 제거
        alias[aliasName.replace(/\/\*$/, "")] = resolve(__dirname, aliasPath);
      }
    }

    return alias;
  } catch (error) {
    console.warn("Failed to load tsconfig paths:", error);
    return {};
  }
}

export default defineConfig({
  resolve: {
    alias: getTsconfigPaths(), // 자동으로 tsconfig paths 적용
  },
});
```

## 장점

### 🚀 개발 경험 향상

- **빠른 Hot Reload**: 소스코드 변경 시 즉시 반영
- **실시간 타입 체킹**: TypeScript가 소스코드를 직접 분석
- **디버깅 편의성**: 원본 소스코드에서 직접 디버깅 가능

### 🔄 이중 참조 전략

- **개발 모드**: `@heart-re-up/hooks` → `../../packages/hooks/src/index.ts` (소스코드 직접 참조)
- **빌드 모드**: `@heart-re-up/hooks` → `./dist/index.js` (package.json exports 사용)

### 📦 DRY 원칙 준수

- 경로 설정을 `tsconfig.json` 한 곳에서만 관리
- Vite 설정이 자동으로 TypeScript 설정을 따름
- 새로운 패키지 추가 시 tsconfig.json만 수정하면 됨

## 설정 예시

### package.json (라이브러리)

```json
{
  "name": "@heart-re-up/hooks",
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    }
  }
}
```

### package.json (앱)

```json
{
  "dependencies": {
    "@heart-re-up/hooks": "workspace:*"
  }
}
```

### 작동 방식

1. **개발 시**: Vite가 alias를 통해 소스코드 직접 참조
2. **빌드 시**: Rollup이 package.json exports를 통해 빌드된 파일 참조
3. **타입 체킹**: TypeScript가 paths를 통해 소스코드 타입 체킹

## 추가 최적화

### 1. 번들 최적화

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["react", "react-dom"],
          "workspace-libs": ["@heart-re-up/hooks"], // 워크스페이스 라이브러리 별도 청크
        },
      },
    },
  },
});
```

### 2. 타입 체크 최적화

```json
// tsconfig.json
{
  "compilerOptions": {
    "composite": true, // 프로젝트 참조 최적화
    "incremental": true // 증분 컴파일
  }
}
```

### 3. 개발 서버 최적화

```typescript
// vite.config.ts
export default defineConfig({
  server: {
    fs: {
      // 워크스페이스 외부 파일 접근 허용
      allow: [".."],
    },
  },
  optimizeDeps: {
    // 워크스페이스 패키지를 사전 번들링에서 제외
    exclude: ["@heart-re-up/hooks"],
  },
});
```

## 모범 사례

### 1. 단일 소스 오브 트루스

- 모든 경로 설정을 `tsconfig.json`에서 관리
- Vite, Jest, 기타 도구들이 이를 자동으로 읽도록 설정

### 2. 환경별 최적화

- 개발: 소스코드 직접 참조로 빠른 피드백
- 빌드: 최적화된 번들로 성능 향상
- 테스트: 소스코드 기반으로 정확한 커버리지

### 3. 확장성 고려

- 새로운 패키지 추가 시 tsconfig.json만 수정
- 자동화된 설정으로 휴먼 에러 방지

이러한 설정을 통해 모노레포의 진정한 장점을 누릴 수 있습니다.
