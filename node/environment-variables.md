# Node.js 환경 변수 관리 가이드

Node.js 서버 사이드 환경 변수 설정, 타입 정의, 검증에 대한 가이드입니다.

> ** 참고**: 클라이언트 사이드(Vite) 환경 변수는 [vite/vite-env.md](../vite/vite-env.md)를 참고하세요.

## 서버 사이드 vs 클라이언트 사이드

| 구분      | 서버 사이드 (Node.js)      | 클라이언트 사이드 (Vite)  |
| --------- | -------------------------- | ------------------------- |
| 접근 범위 | 서버에서만                 | 브라우저에서도 접근       |
| 보안 수준 | 높음 (민감 정보 포함 가능) | 낮음 (공개 정보만)        |
| 접두사    | 제한 없음                  | `VITE_` 필요              |
| 사용 예시 | DB 연결, API 키            | 공개 API URL, 기능 플래그 |

## 기본 환경 변수 설정

### .env.example (템플릿)

```bash
# Node.js 환경
NODE_ENV=development
PORT=3000

# 데이터베이스
DATABASE_URL=postgresql://localhost:5432/myapp
REDIS_URL=redis://localhost:6379

# JWT 및 보안
JWT_SECRET=your-secret-key-minimum-32-characters
SESSION_SECRET=your-session-secret-key

# 외부 서비스 (서버 사이드)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=
SMTP_PASS=

# API 키 (서버 사이드)
STRIPE_SECRET_KEY=
SENDGRID_API_KEY=
OPENAI_API_KEY=

# 서버 설정
CORS_ORIGIN=http://localhost:3000
API_RATE_LIMIT=100
```

### 환경별 설정 파일

```
project/
├── .env.example          # 템플릿 파일
├── .env.development      # 개발 환경
├── .env.staging         # 스테이징 환경
├── .env.production      # 프로덕션 환경
└── .env.local           # 로컬 개발용 (gitignore)
```

#### .env.development

```bash
# 개발 환경 설정
NODE_ENV=development
PORT=3001

# 로컬 데이터베이스
DATABASE_URL=postgresql://localhost:5432/myapp_dev
REDIS_URL=redis://localhost:6379

# 개발용 JWT 시크릿
JWT_SECRET=development-secret-key-32-chars-minimum
SESSION_SECRET=dev-session-secret

# 개발 도구
LOG_LEVEL=debug
DEBUG=app:*
ENABLE_MOCK_DATA=true
```

#### .env.production

```bash
# 프로덕션 환경 설정
NODE_ENV=production
PORT=8080

# 프로덕션 데이터베이스 (환경 변수로 주입)
DATABASE_URL=${DATABASE_URL}
REDIS_URL=${REDIS_URL}

# 보안 설정
JWT_SECRET=${JWT_SECRET}
SESSION_SECRET=${SESSION_SECRET}

# 로깅
LOG_LEVEL=error
SENTRY_DSN=${SENTRY_DSN}
```

## TypeScript 타입 정의

### 환경 변수 타입 선언

```typescript
// types/environment.d.ts
declare global {
  namespace NodeJS {
    interface ProcessEnv {
      // 기본 환경
      NODE_ENV: "development" | "staging" | "production";
      PORT?: string;

      // 데이터베이스
      DATABASE_URL: string;
      REDIS_URL?: string;

      // 인증 및 보안
      JWT_SECRET: string;
      SESSION_SECRET: string;

      // 외부 서비스
      SMTP_HOST?: string;
      SMTP_PORT?: string;
      SMTP_USER?: string;
      SMTP_PASS?: string;

      // API 키 (서버에서만 사용)
      STRIPE_SECRET_KEY?: string;
      SENDGRID_API_KEY?: string;
      OPENAI_API_KEY?: string;

      // 서버 설정
      CORS_ORIGIN?: string;
      API_RATE_LIMIT?: string;
      LOG_LEVEL?: "debug" | "info" | "warn" | "error";

      // 개발 도구
      DEBUG?: string;
      ENABLE_MOCK_DATA?: string;
      SENTRY_DSN?: string;
    }
  }
}

export {};
```

### tsconfig.json 설정

```json
{
  "compilerOptions": {
    "types": ["node"],
    "typeRoots": ["./types", "./node_modules/@types"]
  },
  "include": ["types/**/*", "src/**/*"]
}
```

## 환경 변수 유효성 검사

### Zod를 사용한 스키마 검증

```typescript
// src/config/environment.ts
import { z } from "zod";

const envSchema = z.object({
  // 기본 환경 설정
  NODE_ENV: z
    .enum(["development", "staging", "production"])
    .default("development"),
  PORT: z
    .string()
    .transform((val) => parseInt(val, 10))
    .refine((val) => val > 0 && val < 65536, "포트는 1-65535 범위여야 합니다")
    .default("3000"),

  // 필수 데이터베이스 설정
  DATABASE_URL: z
    .string()
    .url("유효한 데이터베이스 URL을 입력하세요")
    .refine(
      (url) => url.startsWith("postgresql://"),
      "PostgreSQL URL이어야 합니다"
    ),

  // 선택적 Redis 설정
  REDIS_URL: z.string().url().optional(),

  // 보안 설정 (필수)
  JWT_SECRET: z
    .string()
    .min(32, "JWT_SECRET은 최소 32자 이상이어야 합니다")
    .regex(/^[A-Za-z0-9+/=]+$/, "JWT_SECRET은 Base64 형식이어야 합니다"),

  SESSION_SECRET: z
    .string()
    .min(16, "SESSION_SECRET은 최소 16자 이상이어야 합니다"),

  // 외부 서비스 (선택적)
  SMTP_HOST: z.string().optional(),
  SMTP_PORT: z
    .string()
    .transform((val) => parseInt(val, 10))
    .optional(),
  SMTP_USER: z.string().email().optional(),
  SMTP_PASS: z.string().optional(),

  // API 키들 (선택적이지만 길이 검증)
  STRIPE_SECRET_KEY: z
    .string()
    .regex(/^sk_/, "Stripe Secret Key는 'sk_'로 시작해야 합니다")
    .optional(),
  SENDGRID_API_KEY: z
    .string()
    .regex(/^SG\./, "SendGrid API Key는 'SG.'로 시작해야 합니다")
    .optional(),
  OPENAI_API_KEY: z
    .string()
    .regex(/^sk-/, "OpenAI API Key는 'sk-'로 시작해야 합니다")
    .optional(),

  // 서버 설정
  CORS_ORIGIN: z.string().url().optional(),
  API_RATE_LIMIT: z
    .string()
    .transform((val) => parseInt(val, 10))
    .default("100"),
  LOG_LEVEL: z.enum(["debug", "info", "warn", "error"]).default("info"),

  // 개발 도구
  DEBUG: z.string().optional(),
  ENABLE_MOCK_DATA: z
    .string()
    .transform((val) => val === "true")
    .default("false"),
  SENTRY_DSN: z.string().url().optional(),
});

// 환경 변수 파싱 및 검증
export const env = envSchema.parse(process.env);

// 타입 추출 (다른 모듈에서 사용)
export type Environment = z.infer<typeof envSchema>;

// 런타임 검증 함수
export function validateEnvironment() {
  try {
    envSchema.parse(process.env);
    console.log(" 환경 변수 검증 완료");
    return true;
  } catch (error) {
    console.error(" 환경 변수 검증 실패:");
    if (error instanceof z.ZodError) {
      error.errors.forEach((err) => {
        console.error(`  - ${err.path.join(".")}: ${err.message}`);
      });
    }
    return false;
  }
}
```

### 서버 시작 시 검증

```typescript
// src/index.ts 또는 server.ts
import { validateEnvironment, env } from "./config/environment";

// 서버 시작 전 환경 변수 검증
if (!validateEnvironment()) {
  console.error("환경 변수 검증 실패로 서버를 시작할 수 없습니다.");
  process.exit(1);
}

// 이제 env 객체를 안전하게 사용 가능
console.log(` 서버가 포트 ${env.PORT}에서 시작됩니다`);
console.log(` 로그 레벨: ${env.LOG_LEVEL}`);
console.log(`🗄️  데이터베이스: ${env.DATABASE_URL.split("@")[1]}`); // 민감 정보 제외
```

## 설정 객체 생성

```typescript
// src/config/index.ts
import { env } from "./environment";

export const config = {
  // 서버 설정
  server: {
    port: env.PORT,
    corsOrigin: env.CORS_ORIGIN || "*",
    rateLimit: env.API_RATE_LIMIT,
  },

  // 데이터베이스 설정
  database: {
    url: env.DATABASE_URL,
    redis: env.REDIS_URL,
  },

  // 인증 설정
  auth: {
    jwtSecret: env.JWT_SECRET,
    sessionSecret: env.SESSION_SECRET,
    tokenExpiry: "24h",
  },

  // 외부 서비스
  email: {
    host: env.SMTP_HOST,
    port: env.SMTP_PORT,
    user: env.SMTP_USER,
    password: env.SMTP_PASS,
  },

  // API 키
  apiKeys: {
    stripe: env.STRIPE_SECRET_KEY,
    sendgrid: env.SENDGRID_API_KEY,
    openai: env.OPENAI_API_KEY,
  },

  // 개발 설정
  development: {
    enableMockData: env.ENABLE_MOCK_DATA,
    debug: env.DEBUG,
    logLevel: env.LOG_LEVEL,
  },

  // 모니터링
  monitoring: {
    sentryDsn: env.SENTRY_DSN,
  },
} as const;

export default config;
```

## 보안 고려사항

### .gitignore 설정

```gitignore
# 환경 변수 파일
.env
.env.local
.env.*.local

# 허용할 파일들 (템플릿)
!.env.example
!.env.development
!.env.test

# 민감한 설정 파일
.env.production.local
.env.staging.local

# 보안 관련 파일
private-key.pem
*.p12
*.key
```

### 민감한 정보 마스킹

```typescript
// src/utils/logger.ts
export function maskSensitiveEnv(env: Record<string, string>) {
  const sensitiveKeys = [
    "SECRET",
    "KEY",
    "TOKEN",
    "PASSWORD",
    "PASS",
    "API_KEY",
    "PRIVATE",
    "CREDENTIAL",
  ];

  return Object.entries(env).reduce((acc, [key, value]) => {
    const isSensitive = sensitiveKeys.some((keyword) =>
      key.toUpperCase().includes(keyword)
    );

    acc[key] = isSensitive ? "***" : value;
    return acc;
  }, {} as Record<string, string>);
}

// 로깅 시 사용
console.log("환경 변수:", maskSensitiveEnv(process.env));
```

### 프로덕션 배포 시 주의사항

1. **환경 변수 주입**: Docker/K8s에서 환경 변수로 주입
2. **시크릿 관리**: AWS Secrets Manager, HashiCorp Vault 등 사용
3. **최소 권한**: 필요한 권한만 부여
4. **로테이션**: 정기적인 시크릿 교체

## 모범 사례

1. **네이밍 컨벤션**: 일관된 환경 변수 명명 규칙
2. **기본값 제공**: 개발 환경을 위한 합리적인 기본값
3. **검증 강화**: 런타임에서 환경 변수 유효성 검사
4. **문서화**: .env.example로 필요한 변수 명시
5. **분리**: 환경별로 설정 파일 분리
6. **보안**: 민감한 정보는 별도 관리 시스템 사용

## 관련 문서

- [vite/vite-env.md](../vite/vite-env.md) - 클라이언트 사이드 환경 변수
- [scripts.md](./scripts.md) - 환경별 스크립트 실행
- [version-management.md](./version-management.md) - Node.js 버전 관리
