# Vite 기본 설정 가이드

Vite의 일반적인 설정, 경로 별칭(alias), 그리고 유명한 플러그인들에 대한 가이드입니다.

## 기본 설정 구조

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { resolve } from "node:path";

export default defineConfig({
  plugins: [react()],
  base: "/",
  resolve: {
    alias: {
      "@": resolve(__dirname, "src"),
    },
  },
  server: {
    port: 3000,
    open: true,
  },
});
```

## 경로 별칭 (Path Alias) 설정

### 기본 별칭 설정

```typescript
// vite.config.ts
export default defineConfig({
  resolve: {
    alias: {
      "@": resolve(__dirname, "src"),
      "@assets": resolve(__dirname, "src/assets"),
      "@components": resolve(__dirname, "src/components"),
      "@hooks": resolve(__dirname, "src/hooks"),
      "@utils": resolve(__dirname, "src/utils"),
      "@types": resolve(__dirname, "src/types"),
    },
  },
});
```

### 모노레포 환경에서의 별칭

```typescript
// vite.config.ts
export default defineConfig({
  resolve: {
    alias: [
      {
        find: "@",
        replacement: resolve(__dirname, "src"),
      },
      {
        find: "@project/ui-component",
        replacement: resolve(__dirname, "../ui-component/src"),
      },
      {
        find: "@project/ui-resource/assets",
        replacement: resolve(__dirname, "../ui-resource/assets"),
      },
      {
        find: "@aia/react-lib",
        replacement: resolve(__dirname, "../react-lib/dist"),
      },
    ],
  },
});
```

### TSConfig와 Vite Alias 동기화

**⚠️ 중요**: Vite와 TypeScript 모두에서 경로 별칭을 설정해야 합니다.

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@assets/*": ["src/assets/*"],
      "@components/*": ["src/components/*"],
      "@project/ui-component/*": ["../ui-component/src/*"],
      "@project/ui-resource/assets/*": ["../ui-resource/assets/*"]
    }
  }
}
```

**동기화가 필요한 이유:**

- Vite: 런타임 번들링과 개발 서버에서 사용
- TSConfig: TypeScript 컴파일러와 IDE 지원에서 사용
- 둘 중 하나만 설정하면 빌드 오류나 IDE 경고가 발생할 수 있음

## 플러그인 설정

Vite는 풍부한 플러그인 생태계를 제공합니다. 플러그인별 상세 설정과 사용법은 별도 문서를 참고하세요.

### 📦 [Vite 플러그인 가이드](./vite-plugins.md)

주요 플러그인들과 설정 방법:

- **React 지원**: @vitejs/plugin-react, @vitejs/plugin-react-swc
- **정적 파일**: vite-plugin-static-copy
- **SVG 처리**: vite-plugin-svgr, vite-plugin-svg-icons
- **번들 분석**: rollup-plugin-visualizer, vite-bundle-analyzer
- **개발 도구**: vite-plugin-eslint, vite-plugin-checker
- **최적화**: @vitejs/plugin-legacy, vite-plugin-pwa
- **커스텀 플러그인 개발** 가이드

### 기본 플러그인 설정 예시

```typescript
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [
    react({
      fastRefresh: true,
      include: "**/*.{js,jsx,ts,tsx}",
    }),
  ],
});
```

## 고급 설정

### 조건부 설정

```typescript
export default defineConfig(({ command, mode }) => {
  const isDev = command === "serve";
  const isProd = mode === "production";

  return {
    plugins: [
      react(),
      // 개발 환경에서만 eslint 실행
      ...(isDev ? [eslint()] : []),
      // 프로덕션에서만 번들 분석
      ...(isProd ? [visualizer()] : []),
    ],
    build: {
      sourcemap: isDev,
      minify: isProd ? "esbuild" : false,
    },
  };
});
```

### 프록시 설정

```typescript
export default defineConfig({
  server: {
    proxy: {
      "/api": {
        target: "http://localhost:8080",
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ""),
        secure: false,
        ws: true, // WebSocket 지원
      },
    },
  },
});
```

### 컨텍스트 경로 설정

```typescript
export default defineConfig(({ mode }) => {
  const contextPath = mode === "production" ? "/myapp" : "/";

  // 환경변수로 설정하여 애플리케이션에서 사용
  process.env.VITE_BASE_URL = contextPath;

  return {
    base: contextPath,
    server: {
      port: 3000,
    },
    preview: {
      port: 3000,
    },
  };
});
```

## 성능 최적화 팁

### 1. 의존성 사전 번들링 설정

```typescript
export default defineConfig({
  optimizeDeps: {
    include: ["react", "react-dom"],
    exclude: ["@my/custom-package"],
  },
});
```

### 2. 청크 분할

```typescript
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["react", "react-dom"],
          ui: ["@mui/material", "@mui/icons-material"],
        },
      },
    },
  },
});
```

### 3. 빌드 성능 향상

```typescript
export default defineConfig({
  build: {
    target: "es2015", // 타겟 브라우저에 맞게 설정
    minify: "esbuild", // 빠른 minify
    rollupOptions: {
      // 외부 의존성 제외 (CDN 사용 시)
      external: ["react", "react-dom"],
    },
  },
});
```

## 모범 사례

1. **별칭 일관성**: Vite와 TSConfig 별칭을 동일하게 유지
2. **플러그인 순서**: 플러그인 순서가 중요할 수 있으므로 문서 확인
3. **환경별 설정**: 개발/프로덕션 환경에 맞는 최적화 적용
4. **캐시 활용**: 가능한 플러그인들의 캐시 옵션 활용
5. **번들 크기 모니터링**: 정기적으로 번들 분석 도구 사용

## 문제 해결

### 자주 발생하는 이슈

1. **별칭이 작동하지 않음**
   - TSConfig paths와 Vite alias 모두 설정했는지 확인
   - baseUrl 설정 확인

2. **HMR이 작동하지 않음**
   - 파일 확장자 확인 (.tsx, .ts)
   - Fast Refresh 설정 확인

3. **빌드 후 경로 오류**
   - base 설정과 실제 배포 경로 일치 확인
   - 정적 자산 경로 확인

## 참고 자료

- [Vite 공식 문서](https://vitejs.dev/)
- [Vite 플러그인 목록](https://github.com/vitejs/awesome-vite)
- [Rollup 플러그인 호환성](https://vite.dev/guide/api-plugin.html#rollup-plugin-compatibility)
