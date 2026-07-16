# Vite 환경 변수 설정 가이드

Vite 프로젝트의 환경 변수 관리와 설정에 대한 가이드입니다.

## 환경 변수 기본 개념

Vite는 `VITE_` 접두사가 붙은 환경 변수만 클라이언트 번들에 노출합니다.

### 환경 변수 접두사

```typescript
// vite.config.ts
export default defineConfig({
  // 기본값: ['VITE_']
  envPrefix: ["VITE_", "REACT_APP_", "PUBLIC_"],
});
```

## 환경 파일 구성

### .env.example (템플릿)

```bash
# Application
NODE_ENV=development
PORT=3000

# API 설정
VITE_API_URL=http://localhost:8080
VITE_API_TIMEOUT=30000

# Feature Flags
VITE_ENABLE_ANALYTICS=false
VITE_ENABLE_DEBUG=true

# External Services
VITE_SENTRY_DSN=
VITE_GA_TRACKING_ID=
VITE_STRIPE_PUBLIC_KEY=

# 내부 설정 (번들에 포함되지 않음)
DATABASE_URL=postgresql://localhost:5432/myapp
JWT_SECRET=your-secret-key
```

### 환경별 설정 파일

#### .env.development

```bash
# 개발 환경 설정
VITE_API_URL=http://localhost:8080
VITE_ENABLE_DEBUG=true
VITE_MOCK_API=true
VITE_LOG_LEVEL=debug

# 개발 도구
VITE_ENABLE_DEVTOOLS=true
VITE_ENABLE_MSW=true
```

#### .env.staging

```bash
# 스테이징 환경 설정
VITE_API_URL=https://staging-api.example.com
VITE_ENABLE_DEBUG=true
VITE_MOCK_API=false
VITE_LOG_LEVEL=warn

# 스테이징 전용
VITE_SENTRY_DSN=https://staging-dsn@sentry.io/project
```

#### .env.production

```bash
# 프로덕션 환경 설정
VITE_API_URL=https://api.example.com
VITE_ENABLE_DEBUG=false
VITE_MOCK_API=false
VITE_LOG_LEVEL=error

# 프로덕션 서비스
VITE_SENTRY_DSN=https://prod-dsn@sentry.io/project
VITE_GA_TRACKING_ID=UA-XXXXXXXXX-X
```

#### .env.local (개발자 개인 설정)

```bash
# 개인 개발 환경 설정 (gitignore에 추가)
VITE_API_URL=http://localhost:3001
VITE_DEBUG_USER_ID=123
VITE_ENABLE_EXPERIMENTAL_FEATURES=true
```

## 환경별 빌드 스크립트

### package.json 스크립트

```json
{
  "scripts": {
    "dev": "vite --mode development",
    "dev:staging": "vite --mode staging",
    "dev:production": "vite --mode production",

    "build": "vite build",
    "build:dev": "vite build --mode development",
    "build:staging": "vite build --mode staging",
    "build:prod": "vite build --mode production",

    "preview": "vite preview",
    "preview:staging": "vite preview --mode staging"
  }
}
```

### 모드별 커스터마이징

```typescript
// vite.config.ts
export default defineConfig(({ mode }) => {
  // .env 파일 로드
  const env = loadEnv(mode, process.cwd(), "");

  return {
    define: {
      __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
      __BUILD_TIME__: JSON.stringify(new Date().toISOString()),
      __MODE__: JSON.stringify(mode),
    },

    server: {
      port: parseInt(env.VITE_DEV_PORT) || 3000,
    },

    build: {
      sourcemap: mode === "development",
      minify: mode === "production" ? "esbuild" : false,
    },
  };
});
```

## TypeScript 타입 정의

### 환경 변수 타입 선언

```typescript
// types/env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  // API 설정
  readonly VITE_API_URL: string;
  readonly VITE_API_TIMEOUT: string;

  // Feature Flags
  readonly VITE_ENABLE_ANALYTICS: string;
  readonly VITE_ENABLE_DEBUG: string;
  readonly VITE_MOCK_API: string;

  // External Services
  readonly VITE_SENTRY_DSN?: string;
  readonly VITE_GA_TRACKING_ID?: string;
  readonly VITE_STRIPE_PUBLIC_KEY?: string;

  // 로깅
  readonly VITE_LOG_LEVEL: string;

  // 개발 도구
  readonly VITE_ENABLE_DEVTOOLS?: string;
  readonly VITE_ENABLE_MSW?: string;
}

interface ImportMeta {
  readonly env: ImportMetaEnv;
}
```

### 컴파일 타임 상수

```typescript
// types/global.d.ts
declare const __APP_VERSION__: string;
declare const __BUILD_TIME__: string;
declare const __MODE__: string;
declare const __DEV__: boolean;
declare const __PROD__: boolean;
```

## 환경 변수 유효성 검사

### Zod를 사용한 검증

```typescript
// src/config/env.ts
import { z } from "zod";

const envSchema = z.object({
  VITE_API_URL: z.string().url("유효한 API URL을 입력하세요"),
  VITE_API_TIMEOUT: z.string().transform((val) => {
    const num = Number(val);
    if (isNaN(num) || num <= 0) {
      throw new Error("API 타임아웃은 양수여야 합니다");
    }
    return num;
  }),

  // Boolean 환경 변수
  VITE_ENABLE_ANALYTICS: z.string().transform((val) => val === "true"),
  VITE_ENABLE_DEBUG: z.string().transform((val) => val === "true"),
  VITE_MOCK_API: z.string().transform((val) => val === "true"),

  // 선택적 환경 변수
  VITE_SENTRY_DSN: z.string().url().optional(),
  VITE_GA_TRACKING_ID: z.string().optional(),

  // Enum 환경 변수
  VITE_LOG_LEVEL: z.enum(["debug", "info", "warn", "error"]).default("info"),
});

// 환경 변수 파싱 및 검증
try {
  export const env = envSchema.parse(import.meta.env);
} catch (error) {
  console.error("❌ 환경 변수 검증 실패:", error);
  throw error;
}

// 타입 안전한 환경 변수 사용
export const config = {
  api: {
    url: env.VITE_API_URL,
    timeout: env.VITE_API_TIMEOUT,
  },
  features: {
    analytics: env.VITE_ENABLE_ANALYTICS,
    debug: env.VITE_ENABLE_DEBUG,
    mockApi: env.VITE_MOCK_API,
  },
  services: {
    sentryDsn: env.VITE_SENTRY_DSN,
    gaTrackingId: env.VITE_GA_TRACKING_ID,
  },
  logging: {
    level: env.VITE_LOG_LEVEL,
  },
} as const;
```

### 런타임 검증

```typescript
// src/config/env-validator.ts
export function validateEnvironment() {
  const requiredEnvs = ["VITE_API_URL", "VITE_API_TIMEOUT"];

  const missing = requiredEnvs.filter((env) => !import.meta.env[env]);

  if (missing.length > 0) {
    throw new Error(`필수 환경 변수가 누락되었습니다: ${missing.join(", ")}`);
  }

  // URL 유효성 검사
  try {
    new URL(import.meta.env.VITE_API_URL);
  } catch {
    throw new Error("VITE_API_URL이 유효한 URL이 아닙니다");
  }

  console.log("✅ 환경 변수 검증 완료");
}

// main.tsx에서 호출
validateEnvironment();
```

## 고급 패턴

### 환경별 기능 제어

```typescript
// src/utils/feature-flags.ts
import { config } from "@/config/env";

export const featureFlags = {
  analytics: config.features.analytics,
  debug: config.features.debug,
  mockApi: config.features.mockApi,

  // 복합 조건
  devtools: config.features.debug && import.meta.env.DEV,

  // 환경별 조건
  enableExperimentalFeatures: import.meta.env.MODE === "development",
} as const;

// 사용 예시
if (featureFlags.analytics) {
  // 분석 도구 초기화
}

if (featureFlags.devtools) {
  // 개발 도구 활성화
}
```

### 환경별 API 엔드포인트

```typescript
// src/config/api.ts
import { config } from "@/config/env";

const createApiConfig = () => {
  const baseURL = config.api.url;
  const timeout = config.api.timeout;

  // 환경별 기본 설정
  const defaultConfig = {
    baseURL,
    timeout,
    headers: {
      "Content-Type": "application/json",
    },
  };

  // 개발 환경 전용 설정
  if (import.meta.env.DEV) {
    return {
      ...defaultConfig,
      headers: {
        ...defaultConfig.headers,
        "X-Debug": "true",
      },
    };
  }

  // 프로덕션 환경 전용 설정
  if (import.meta.env.PROD) {
    return {
      ...defaultConfig,
      headers: {
        ...defaultConfig.headers,
        "X-App-Version": __APP_VERSION__,
      },
    };
  }

  return defaultConfig;
};

export const apiConfig = createApiConfig();
```

### 조건부 모듈 로딩

```typescript
// src/config/conditional-imports.ts
import { featureFlags } from "@/utils/feature-flags";

// Mock Service Worker (개발 환경에서만)
export const setupMocks = async () => {
  if (featureFlags.mockApi && import.meta.env.DEV) {
    const { worker } = await import("../mocks/browser");
    return worker.start();
  }
};

// 분석 도구 (프로덕션에서만)
export const setupAnalytics = async () => {
  if (featureFlags.analytics && import.meta.env.PROD) {
    const { initAnalytics } = await import("../analytics");
    return initAnalytics(config.services.gaTrackingId);
  }
};

// 에러 추적 (프로덕션에서만)
export const setupErrorTracking = async () => {
  if (config.services.sentryDsn && import.meta.env.PROD) {
    const { init } = await import("@sentry/react");
    return init({
      dsn: config.services.sentryDsn,
      environment: import.meta.env.MODE,
    });
  }
};
```

## 보안 고려사항

### 민감한 정보 처리

```typescript
// ❌ 잘못된 예시 - 클라이언트에 노출됨
const VITE_API_SECRET = "secret-key"; // 위험!

// ✅ 올바른 예시 - 서버에서만 사용
const API_SECRET = "secret-key"; // VITE_ 접두사 없음

// 환경 변수 마스킹
export function maskSensitiveEnv(env: Record<string, string>) {
  const sensitiveKeys = ["SECRET", "KEY", "TOKEN", "PASSWORD"];

  return Object.entries(env).reduce((acc, [key, value]) => {
    const isSensitive = sensitiveKeys.some((keyword) =>
      key.toUpperCase().includes(keyword)
    );

    acc[key] = isSensitive ? "***" : value;
    return acc;
  }, {} as Record<string, string>);
}
```

### .gitignore 설정

```gitignore
# 환경 변수 파일
.env
.env.local
.env.*.local

# 보안 파일
.env.production.local
.env.staging.local

# 허용할 파일들
!.env.example
!.env.development
!.env.test
```

## 디버깅 도구

### 환경 변수 검사기

```typescript
// src/utils/env-inspector.ts
export function inspectEnvironment() {
  if (!import.meta.env.DEV) return;

  console.group("🔧 Environment Inspector");

  console.log("Mode:", import.meta.env.MODE);
  console.log("Dev:", import.meta.env.DEV);
  console.log("Prod:", import.meta.env.PROD);

  console.log("\n📝 Environment Variables:");
  Object.entries(import.meta.env)
    .filter(([key]) => key.startsWith("VITE_"))
    .forEach(([key, value]) => {
      console.log(`${key}:`, value);
    });

  console.log("\n⚙️ Build Constants:");
  console.log("Version:", __APP_VERSION__);
  console.log("Build Time:", __BUILD_TIME__);

  console.groupEnd();
}

// 개발 환경에서만 실행
if (import.meta.env.DEV) {
  inspectEnvironment();
}
```

## 모범 사례

### 1. 명명 규칙

```bash
# 카테고리별 그룹화
VITE_API_URL=
VITE_API_TIMEOUT=
VITE_API_VERSION=

VITE_FEATURE_ANALYTICS=
VITE_FEATURE_DEBUG=
VITE_FEATURE_MOCK_API=

VITE_SERVICE_SENTRY_DSN=
VITE_SERVICE_GA_ID=
```

### 2. 기본값 설정

```typescript
// src/config/defaults.ts
export const defaultConfig = {
  api: {
    url: import.meta.env.VITE_API_URL || "http://localhost:3001",
    timeout: Number(import.meta.env.VITE_API_TIMEOUT) || 30000,
  },
  features: {
    debug: import.meta.env.VITE_ENABLE_DEBUG === "true" || import.meta.env.DEV,
  },
} as const;
```

### 3. 환경 변수 문서화

```typescript
// env.d.ts에 JSDoc 추가
interface ImportMetaEnv {
  /** API 서버 기본 URL */
  readonly VITE_API_URL: string;

  /** API 요청 타임아웃 (밀리초) */
  readonly VITE_API_TIMEOUT: string;

  /** 분석 도구 활성화 여부 */
  readonly VITE_ENABLE_ANALYTICS: string;
}
```

## 참고 자료

- [Vite 환경 변수 문서](https://vitejs.dev/guide/env-and-mode.html)
- [Zod 스키마 검증](https://zod.dev/)
- [환경 변수 보안 가이드](https://12factor.net/config)

## 관련 문서

- [../typescript.md](../typescript.md) - TypeScript 설정
- [./vite-dev.md](./vite-dev.md) - 개발서버 설정
- [../node/environment-variables.md](../node/environment-variables.md) - 서버사이드 환경 변수
- [../quality/README.md](../quality/README.md) - 코드 품질 설정
