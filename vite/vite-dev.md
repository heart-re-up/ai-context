# Vite 개발서버 설정 가이드

React 애플리케이션의 개발 환경 구성과 개발서버 최적화에 대한 가이드입니다.

## 기본 개발서버 설정

### 기본 구성

```typescript
// vite.config.ts
export default defineConfig({
  server: {
    port: 3000,
    host: true, // 모든 네트워크 인터페이스에서 접근 가능
    open: true, // 서버 시작 시 브라우저 자동 열기
    cors: true, // CORS 활성화
    strictPort: false, // 포트가 이미 사용 중이면 다른 포트 사용
  },
  preview: {
    port: 4173,
    host: true,
    open: true,
  },
});
```

### 환경별 포트 설정

```typescript
// vite.config.ts
export default defineConfig(({ mode }) => {
  const serverConfig = {
    development: { port: 8001 },
    staging: { port: 8002 },
    production: { port: 8003 },
  };

  const config = serverConfig[mode] || serverConfig.development;

  return {
    server: {
      ...config,
      hmr: {
        overlay: false, // 에러 오버레이 비활성화
      },
    },
    preview: {
      ...config,
    },
  };
});
```

## HMR (Hot Module Replacement) 설정

### 기본 HMR 설정

```typescript
export default defineConfig({
  server: {
    hmr: {
      overlay: true, // 에러 오버레이 표시
      port: 24678, // HMR 전용 포트 (선택사항)
    },
  },
});
```

### HMR 문제 해결

```typescript
export default defineConfig({
  server: {
    hmr: {
      overlay: false, // 오버레이가 방해되는 경우
      clientPort: 443, // HTTPS 환경에서 필요한 경우
    },
    // 파일 감시 설정
    watch: {
      usePolling: true, // Docker나 VM 환경에서 필요
      interval: 1000,
    },
  },
});
```

### React Fast Refresh 최적화

```typescript
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [
    react({
      fastRefresh: true,
      // Babel 설정으로 추가 최적화
      babel: {
        plugins: [
          // 개발 중 유용한 플러그인들
          [
            "@babel/plugin-transform-react-jsx-development",
            { runtime: "automatic" },
          ],
        ],
      },
    }),
  ],
});
```

## 프록시 설정

### API 프록시

```typescript
export default defineConfig({
  server: {
    proxy: {
      // 기본 API 프록시
      "/api": {
        target: "http://localhost:8080",
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ""),
        // SSL 인증서 검증 무시 (개발 환경)
        secure: false,
        // 요청/응답 로깅
        configure: (proxy, options) => {
          proxy.on("proxyReq", (proxyReq, req, res) => {
            console.log("→", req.method, req.url);
          });
          proxy.on("proxyRes", (proxyRes, req, res) => {
            console.log("←", proxyRes.statusCode, req.url);
          });
        },
      },
      // WebSocket 프록시
      "/ws": {
        target: "ws://localhost:8080",
        ws: true,
        changeOrigin: true,
      },
      // 정적 파일 프록시
      "/assets": {
        target: "http://localhost:8080",
        changeOrigin: true,
      },
    },
  },
});
```

### 조건부 프록시 설정

```typescript
export default defineConfig(({ mode }) => {
  const apiTargets = {
    development: "http://localhost:8080",
    staging: "https://staging-api.example.com",
    production: "https://api.example.com",
  };

  const target = apiTargets[mode] || apiTargets.development;

  return {
    server: {
      proxy: {
        "/api": {
          target,
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, ""),
          secure: mode === "production",
        },
      },
    },
  };
});
```

## 환경 변수 관리

### 개발 환경 변수

```typescript
// .env.development
VITE_API_URL=http://localhost:8080
VITE_APP_TITLE=MyApp Development
VITE_DEBUG=true

// .env.local (gitignore에 추가)
VITE_SECRET_KEY=dev-secret-key
```

### 환경별 설정

```typescript
// vite.config.ts
export default defineConfig(({ mode }) => {
  // 환경 파일 로드
  const env = loadEnv(mode, process.cwd(), "");

  return {
    define: {
      __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
      __BUILD_TIME__: JSON.stringify(new Date().toISOString()),
    },
    server: {
      port: parseInt(env.VITE_DEV_PORT) || 3000,
    },
  };
});
```

## 개발 도구 통합

### ESLint 통합

```typescript
import eslint from "vite-plugin-eslint";

export default defineConfig({
  plugins: [
    react(),
    eslint({
      cache: false,
      include: ["src/**/*.{js,jsx,ts,tsx}"],
      exclude: ["node_modules", "**/*.d.ts"],
      failOnWarning: false,
      failOnError: false, // 개발 중에는 에러로 인한 빌드 중단 방지
    }),
  ],
});
```

### 타입 체크 통합

```typescript
import checker from "vite-plugin-checker";

export default defineConfig({
  plugins: [
    react(),
    checker({
      typescript: true,
      eslint: {
        lintCommand: 'eslint "./src/**/*.{ts,tsx}"',
      },
      overlay: {
        initialIsOpen: false, // 오버레이 기본적으로 닫힌 상태
      },
    }),
  ],
});
```

## 성능 최적화

### 개발 빌드 최적화

```typescript
export default defineConfig(({ command }) => {
  const isDev = command === "serve";

  return {
    // 개발 환경에서는 빠른 빌드 우선
    esbuild: isDev
      ? {
          target: "esnext",
          format: "esm",
        }
      : undefined,

    optimizeDeps: {
      // 자주 사용하는 의존성 사전 번들링
      include: [
        "react",
        "react-dom",
        "react-router-dom",
        "@mui/material",
        "lodash",
      ],
      // 느린 의존성 제외
      exclude: ["@storybook/react"],
    },

    server: {
      // 파일 감시 최적화
      fs: {
        strict: false, // 심볼릭 링크 허용
      },
    },
  };
});
```

### 메모리 사용량 최적화

```typescript
export default defineConfig({
  server: {
    watch: {
      // 불필요한 파일 감시 제외
      ignored: [
        "**/.git/**",
        "**/node_modules/**",
        "**/dist/**",
        "**/coverage/**",
        "**/.vscode/**",
      ],
    },
  },
  optimizeDeps: {
    // 의존성 캐시 설정
    force: false, // 강제 재빌드 비활성화
  },
});
```

## 디버깅 설정

### 소스맵 설정

```typescript
export default defineConfig(({ command }) => {
  return {
    build: {
      sourcemap: command === "serve" ? "inline" : false,
    },
    css: {
      devSourcemap: true, // CSS 소스맵 활성화
    },
  };
});
```

### 개발 도구 설정

```typescript
export default defineConfig({
  define: {
    __DEV__: JSON.stringify(true),
  },
  plugins: [
    react({
      // React DevTools 최적화
      babel: {
        plugins: [
          [
            "@babel/plugin-transform-react-jsx-development",
            {
              runtime: "automatic",
            },
          ],
        ],
      },
    }),
  ],
});
```

## 모노레포 개발 환경

### 워크스페이스 간 라이브 리로딩

```typescript
export default defineConfig({
  server: {
    watch: {
      // 다른 워크스페이스 파일도 감시
      include: ["../ui-component/src/**", "../shared/src/**"],
    },
  },
  resolve: {
    alias: {
      "@project/ui-component": resolve(__dirname, "../ui-component/src"),
      "@project/shared": resolve(__dirname, "../shared/src"),
    },
  },
});
```

### 의존성 링크 설정

```typescript
export default defineConfig({
  optimizeDeps: {
    // 로컬 패키지는 번들링하지 않음
    exclude: ["@project/ui-component", "@project/shared"],
    // 외부 의존성만 사전 번들링
    include: ["react", "react-dom"],
  },
});
```

## HTTPS 개발 서버

### 로컬 HTTPS 설정

```typescript
import fs from "fs";

export default defineConfig({
  server: {
    https: {
      key: fs.readFileSync("./certs/localhost-key.pem"),
      cert: fs.readFileSync("./certs/localhost.pem"),
    },
    port: 443,
  },
});
```

### mkcert를 사용한 인증서 생성

```bash
# mkcert 설치 (macOS)
brew install mkcert
brew install nss # Firefox 지원

# 로컬 CA 설치
mkcert -install

# localhost 인증서 생성
mkcert localhost 127.0.0.1 ::1

# Vite 설정에서 사용
```

## 개발 워크플로우 최적화

### 빠른 재시작 스크립트

```json
// package.json
{
  "scripts": {
    "dev": "vite",
    "dev:debug": "DEBUG=vite:* vite",
    "dev:clean": "rm -rf node_modules/.vite && vite",
    "dev:port": "vite --port 3001",
    "dev:host": "vite --host 0.0.0.0"
  }
}
```

### 개발 환경 확인 도구

```typescript
// src/utils/dev.ts
export const isDevelopment = import.meta.env.DEV;
export const isProduction = import.meta.env.PROD;

export function devOnly<T>(value: T): T | undefined {
  return isDevelopment ? value : undefined;
}

// 개발 환경에서만 로그 출력
export const devLog = (...args: any[]) => {
  if (isDevelopment) {
    console.log("[DEV]", ...args);
  }
};
```

## 문제 해결

### 자주 발생하는 개발 환경 이슈

1. **HMR이 작동하지 않음**

   ```typescript
   // 해결 방법
   server: {
     watch: {
       usePolling: true, // 파일 시스템 이슈
     },
     hmr: {
       overlay: false, // 오버레이 간섭
     },
   }
   ```

2. **프록시 설정이 작동하지 않음**

   ```typescript
   // 경로 확인
   proxy: {
     '^/api/.*': { // 정규표현식 사용
       target: 'http://localhost:8080',
       changeOrigin: true,
     },
   }
   ```

3. **메모리 부족 오류**

   ```bash
   # Node.js 메모리 늘리기
   NODE_OPTIONS="--max-old-space-size=4096" npm run dev
   ```

4. **느린 시작 속도**
   ```typescript
   optimizeDeps: {
     force: true, // 의존성 강제 재빌드 (한 번만)
   }
   ```

## 모범 사례

1. **환경 분리**: 개발/스테이징/프로덕션 환경 명확히 분리
2. **프록시 활용**: 백엔드 API와의 CORS 문제 해결
3. **HMR 최적화**: 빠른 피드백 루프 구성
4. **디버깅 도구**: 소스맵과 React DevTools 활용
5. **성능 모니터링**: 번들 크기와 빌드 시간 주기적 확인

## 참고 자료

- [Vite 개발서버 문서](https://vitejs.dev/config/server-options.html)
- [React Fast Refresh](https://github.com/facebook/react/tree/main/packages/react-refresh)
- [Proxy 설정 가이드](https://vitejs.dev/config/server-options.html#server-proxy)
