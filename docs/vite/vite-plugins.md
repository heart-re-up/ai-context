# Vite 플러그인 가이드

Vite 생태계의 다양한 플러그인들과 그 설정 방법에 대한 종합 가이드입니다.

## 개요

Vite는 플러그인 기반 아키텍처를 통해 다양한 기능을 확장할 수 있습니다. Rollup 플러그인과 호환되며, Vite 전용 플러그인도 풍부하게 제공됩니다.

## React 관련 플러그인

### @vitejs/plugin-react

React 프로젝트의 핵심 플러그인으로 JSX 변환과 Fast Refresh를 제공합니다.

```typescript
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [
    react({
      // Fast Refresh 설정
      fastRefresh: true,

      // Babel 설정 커스터마이징
      babel: {
        plugins: [
          "@babel/plugin-proposal-decorators",
          [
            "@babel/plugin-transform-react-jsx-development",
            { runtime: "automatic" },
          ],
        ],
        presets: [["@babel/preset-env", { targets: "defaults" }]],
      },

      // 특정 파일만 포함
      include: "**/*.{js,jsx,ts,tsx}",
      exclude: /node_modules/,

      // JSX pragma 설정
      jsxImportSource: "@emotion/react",
    }),
  ],
});
```

### @vitejs/plugin-react-swc (더 빠른 컴파일)

SWC를 사용하여 더 빠른 컴파일 속도를 제공합니다.

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react-swc";

export default defineConfig({
  plugins: [
    react({
      // SWC 특정 옵션
      jsxImportSource: "@emotion/react",
      plugins: [
        // SWC 플러그인 설정
        ["@swc/plugin-emotion", {}],
      ],
    }),
  ],
});
```

**성능 비교:**

- 일반적으로 Babel보다 3-10배 빠른 컴파일 속도
- 대규모 프로젝트에서 특히 효과적
- Babel 플러그인과의 호환성은 제한적

## 정적 파일 처리

### vite-plugin-static-copy

정적 파일을 빌드 출력 디렉토리로 복사합니다.

```typescript
import { viteStaticCopy } from "vite-plugin-static-copy";

export default defineConfig({
  plugins: [
    viteStaticCopy({
      targets: [
        // 기본 복사
        {
          src: resolve(__dirname, "../ui-resource/public/*"),
          dest: "",
        },
        // 특정 확장자만 복사
        {
          src: "assets/icons/*.{png,jpg,svg}",
          dest: "icons",
        },
        // 중첩 디렉토리 구조 유지
        {
          src: "public/**/*",
          dest: "static",
          flatten: false, // 디렉토리 구조 유지
        },
        // 파일 변환과 함께 복사
        {
          src: "manifest.json",
          dest: "",
          transform: (contents) => {
            const manifest = JSON.parse(contents.toString());
            manifest.version = process.env.npm_package_version;
            return JSON.stringify(manifest, null, 2);
          },
        },
      ],

      // 감시 옵션
      watch: {
        reloadPageOnChange: true,
        paths: ["public/**/*", "assets/**/*"],
      },

      // 구조 옵션
      structured: true, // 디렉토리 구조 유지
      silent: false, // 로그 출력
    }),
  ],
});
```

## SVG 처리

### vite-plugin-svgr

SVG 파일을 React 컴포넌트로 변환합니다.

```typescript
import svgr from "vite-plugin-svgr";

export default defineConfig({
  plugins: [
    svgr({
      // SVGR 옵션
      svgrOptions: {
        icon: true, // viewBox 자동 추가
        typescript: false,
        dimensions: false, // width/height 제거
        expandProps: "end", // props 전개 위치
        svgProps: {
          role: "img",
          "aria-hidden": "true",
        },
        // 플러그인 설정
        plugins: ["@svgr/plugin-jsx", "@svgr/plugin-prettier"],
      },

      // 파일 필터링
      include: "**/*.svg?react",
      exclude: /node_modules/,

      // TypeScript 지원
      typescript: true,
    }),
  ],
});
```

**사용법:**

```typescript
// React 컴포넌트로 import
import Logo from "./logo.svg?react";
import { ReactComponent as Icon } from "./icon.svg";

// 일반 URL로 import
import logoUrl from "./logo.svg";

function App() {
  return (
    <div>
      <Logo className="logo" />
      <Icon width={24} height={24} />
      <img src={logoUrl} alt="Logo" />
    </div>
  );
}
```

### vite-plugin-svg-icons

SVG 스프라이트를 생성하여 성능을 최적화합니다.

```typescript
import { createSvgIconsPlugin } from "vite-plugin-svg-icons";

export default defineConfig({
  plugins: [
    createSvgIconsPlugin({
      iconDirs: [resolve(process.cwd(), "src/assets/icons")],
      symbolId: "icon-[dir]-[name]",
      inject: "body-last",
      customDomId: "__svg__icons__dom__",
    }),
  ],
});
```

## CSS 및 스타일 처리

### vite-plugin-lib-inject-css

라이브러리 빌드 시 CSS를 JS에 인라인으로 주입합니다.

```typescript
import { libInjectCss } from "vite-plugin-lib-inject-css";

export default defineConfig({
  plugins: [
    react(),
    libInjectCss({
      // CSS 주입 옵션
      include: ["**/*.css", "**/*.scss"],
      exclude: /node_modules/,
    }),
  ],
  build: {
    lib: {
      entry: resolve(__dirname, "src/index.ts"),
      formats: ["es", "cjs"],
    },
  },
});
```

### @vitejs/plugin-legacy

레거시 브라우저 지원을 위한 폴리필을 추가합니다.

```typescript
import legacy from "@vitejs/plugin-legacy";

export default defineConfig({
  plugins: [
    legacy({
      targets: ["defaults", "not IE 11"],
      additionalLegacyPolyfills: ["regenerator-runtime/runtime"],
      modernPolyfills: true,
    }),
  ],
});
```

## 번들 분석 및 최적화

### rollup-plugin-visualizer

번들 구성을 시각적으로 분석합니다.

```typescript
import { visualizer } from "rollup-plugin-visualizer";

export default defineConfig({
  plugins: [
    visualizer({
      open: false,
      gzipSize: true,
      brotliSize: true,
      filename: "./dist/bundle-report.html",
      template: "treemap", // sunburst, treemap, network
      sourcemap: true,
    }),
  ],
});
```

### vite-bundle-analyzer

실시간 번들 분석 서버를 제공합니다.

```typescript
import { analyzer } from "vite-bundle-analyzer";

export default defineConfig({
  plugins: [
    analyzer({
      analyzerMode: "server",
      analyzerPort: 8888,
      openAnalyzer: true,
      generateStatsFile: true,
      statsFilename: "bundle-stats.json",
    }),
  ],
});
```

## 개발 도구 플러그인

### vite-plugin-eslint

빌드 중 ESLint 검사를 수행합니다.

```typescript
import eslint from "vite-plugin-eslint";

export default defineConfig({
  plugins: [
    eslint({
      cache: false,
      include: ["src/**/*.{js,jsx,ts,tsx}"],
      exclude: ["node_modules", "**/*.d.ts"],
      failOnWarning: false,
      failOnError: false, // 개발 중에는 에러로 인한 빌드 중단 방지
      formatter: "stylish",

      // ESLint 옵션
      eslintPath: "eslint",
      lintOnStart: true,
      emitWarning: true,
      emitError: true,
    }),
  ],
});
```

### vite-plugin-checker

TypeScript, ESLint 등의 검사를 통합 실행합니다.

```typescript
import checker from "vite-plugin-checker";

export default defineConfig({
  plugins: [
    checker({
      typescript: {
        tsconfigPath: "./tsconfig.json",
        buildMode: true,
      },
      eslint: {
        lintCommand: 'eslint "./src/**/*.{ts,tsx}" --cache',
        dev: {
          logLevel: ["error", "warning"],
        },
      },
      stylelint: {
        lintCommand: 'stylelint "./src/**/*.{css,scss}"',
      },
      overlay: {
        initialIsOpen: false,
        position: "tl", // top-left
        badgeStyle: "margin: 8px;",
      },
      terminal: true,
      enableBuild: false, // 빌드 시에는 비활성화
    }),
  ],
});
```

## 환경 및 설정 플러그인

### vite-plugin-environment

환경 변수를 더 유연하게 처리합니다.

```typescript
export default defineConfig({
  define: {
    __DEV__: JSON.stringify(process.env.NODE_ENV !== "production"),
    __VERSION__: JSON.stringify(process.env.npm_package_version),
    __BUILD_TIME__: JSON.stringify(new Date().toISOString()),

    // 런타임 환경 정보
    global: "globalThis",
  },

  // 환경 변수 접두사 설정
  envPrefix: ["VITE_", "REACT_APP_"],

  // .env 파일 경로 커스터마이징
  envDir: "./env",
});
```

### vite-plugin-mock

개발 중 Mock API를 제공합니다.

```typescript
import { viteMockServe } from "vite-plugin-mock";

export default defineConfig(({ command }) => {
  return {
    plugins: [
      viteMockServe({
        mockPath: "mock",
        localEnabled: command === "serve",
        prodEnabled: false,
        injectCode: `
          import { setupProdMockServer } from '../mock/_createProductionServer';
          setupProdMockServer();
        `,
        logger: true,
        supportTs: true,
      }),
    ],
  };
});
```

## 테스트 관련 플러그인

### @storybook/builder-vite

Storybook과 Vite를 통합합니다.

```typescript
// .storybook/main.ts
import type { StorybookConfig } from "@storybook/react-vite";

const config: StorybookConfig = {
  framework: "@storybook/react-vite",

  viteFinal: async (config, { configType }) => {
    // Vite 설정 커스터마이징
    if (configType === "DEVELOPMENT") {
      config.define = {
        ...config.define,
        __DEV__: true,
      };
    }

    // 별칭 설정 추가
    config.resolve.alias = {
      ...config.resolve.alias,
      "@": resolve(__dirname, "../src"),
    };

    return config;
  },
};

export default config;
```

## PWA 플러그인

### vite-plugin-pwa

Progressive Web App 기능을 추가합니다.

```typescript
import { VitePWA } from "vite-plugin-pwa";

export default defineConfig({
  plugins: [
    VitePWA({
      registerType: "autoUpdate",
      workbox: {
        clientsClaim: true,
        skipWaiting: true,
        globPatterns: ["**/*.{js,css,html,ico,png,svg}"],
      },
      includeAssets: ["favicon.ico", "apple-touch-icon.png"],
      manifest: {
        name: "My App",
        short_name: "MyApp",
        description: "My Awesome App description",
        theme_color: "#ffffff",
        background_color: "#ffffff",
        display: "standalone",
        icons: [
          {
            src: "pwa-192x192.png",
            sizes: "192x192",
            type: "image/png",
          },
          {
            src: "pwa-512x512.png",
            sizes: "512x512",
            type: "image/png",
          },
        ],
      },
    }),
  ],
});
```

## 조건부 플러그인 사용

### 환경별 플러그인 적용

```typescript
export default defineConfig(({ command, mode }) => {
  const isDev = command === "serve";
  const isProd = mode === "production";

  const plugins = [react()];

  // 개발 환경 전용 플러그인
  if (isDev) {
    plugins.push(
      eslint(),
      checker({
        typescript: true,
        eslint: { lintCommand: 'eslint "./src/**/*.{ts,tsx}"' },
      })
    );
  }

  // 프로덕션 환경 전용 플러그인
  if (isProd) {
    plugins.push(
      visualizer({
        filename: "./dist/bundle-report.html",
        open: false,
      }),
      legacy({
        targets: ["defaults", "not IE 11"],
      })
    );
  }

  // 특정 모드에서만 실행
  if (mode === "analyze") {
    plugins.push(
      analyzer({
        analyzerMode: "server",
        openAnalyzer: true,
      })
    );
  }

  return {
    plugins,
    build: {
      sourcemap: isDev,
      minify: isProd ? "esbuild" : false,
    },
  };
});
```

## 커스텀 플러그인 개발

### 간단한 플러그인 예제

```typescript
// plugins/hello-world.ts
import type { Plugin } from "vite";

export function helloWorld(): Plugin {
  return {
    name: "hello-world",
    configResolved(config) {
      console.log("Hello World from Vite plugin!");
    },
    transformIndexHtml(html) {
      return html.replace("{{TITLE}}", "My Awesome App");
    },
    generateBundle(options, bundle) {
      // 번들 생성 시 실행
      this.emitFile({
        type: "asset",
        fileName: "build-info.json",
        source: JSON.stringify({
          buildTime: new Date().toISOString(),
          mode: process.env.NODE_ENV,
        }),
      });
    },
  };
}

// vite.config.ts에서 사용
import { helloWorld } from "./plugins/hello-world";

export default defineConfig({
  plugins: [react(), helloWorld()],
});
```

### 고급 플러그인 패턴

```typescript
import type { Plugin, ResolvedConfig } from "vite";

interface AdvancedPluginOptions {
  include?: string[];
  exclude?: string[];
  transformOptions?: Record<string, any>;
}

export function advancedPlugin(options: AdvancedPluginOptions = {}): Plugin {
  let config: ResolvedConfig;

  return {
    name: "advanced-plugin",
    configResolved(resolvedConfig) {
      config = resolvedConfig;
    },

    resolveId(id) {
      // 모듈 해결 로직
      if (id === "virtual:my-module") {
        return id;
      }
    },

    load(id) {
      // 가상 모듈 로드
      if (id === "virtual:my-module") {
        return `export const config = ${JSON.stringify(config.env)};`;
      }
    },

    transform(code, id) {
      // 코드 변환 로직
      if (options.include?.some((pattern) => id.includes(pattern))) {
        return {
          code: `// Transformed by advanced-plugin\n${code}`,
          map: null,
        };
      }
    },
  };
}
```

## 플러그인 최적화 팁

### 1. 플러그인 순서

```typescript
export default defineConfig({
  plugins: [
    // 1. 소스 변환 플러그인들
    react(),
    svgr(),

    // 2. 정적 파일 처리
    viteStaticCopy(),

    // 3. 개발 도구 (개발 환경에서만)
    ...(isDev ? [eslint(), checker()] : []),

    // 4. 분석 도구 (마지막에)
    ...(isProd ? [visualizer()] : []),
  ],
});
```

### 2. 플러그인 캐싱

```typescript
export default defineConfig({
  plugins: [
    eslint({
      cache: true, // 캐시 활성화
      cacheLocation: "node_modules/.cache/eslint",
    }),
    checker({
      typescript: {
        buildMode: false, // 개발 모드에서는 빠른 체크
      },
    }),
  ],
});
```

### 3. 조건부 로딩

```typescript
export default defineConfig(async ({ command, mode }) => {
  const plugins = [react()];

  // 필요할 때만 플러그인 동적 로드
  if (command === "serve") {
    const { default: eslint } = await import("vite-plugin-eslint");
    plugins.push(eslint());
  }

  return { plugins };
});
```

## 문제 해결

### 자주 발생하는 플러그인 이슈

1. **플러그인 충돌**

   ```typescript
   // 플러그인 순서 조정으로 해결
   plugins: [
     react(), // React 플러그인을 먼저
     svgr(), // 그 다음 SVGR
   ];
   ```

2. **TypeScript 타입 오류**

   ```typescript
   // vite-env.d.ts에 타입 선언 추가
   /// <reference types="vite/client" />
   /// <reference types="vite-plugin-svgr/client" />
   ```

3. **HMR이 작동하지 않는 경우**

   ```typescript
   // include/exclude 패턴 확인
   react({
     include: "**/*.{js,jsx,ts,tsx}",
     exclude: /node_modules/,
   });
   ```

4. **번들 크기 증가**
   ```typescript
   // 불필요한 플러그인 제거 또는 조건부 적용
   ...(process.env.ANALYZE === "true" ? [visualizer()] : [])
   ```

## 플러그인 생태계

### 인기 플러그인 목록

| 플러그인                 | 용도           | 설치                             |
| ------------------------ | -------------- | -------------------------------- |
| @vitejs/plugin-react     | React 지원     | `npm i @vitejs/plugin-react`     |
| vite-plugin-svgr         | SVG → React    | `npm i vite-plugin-svgr`         |
| rollup-plugin-visualizer | 번들 분석      | `npm i rollup-plugin-visualizer` |
| vite-plugin-eslint       | ESLint 통합    | `npm i vite-plugin-eslint`       |
| vite-plugin-checker      | 타입/린트 체크 | `npm i vite-plugin-checker`      |
| vite-plugin-pwa          | PWA 지원       | `npm i vite-plugin-pwa`          |

### 플러그인 검색 및 선택 가이드

1. **공식 플러그인 우선**: `@vitejs/` 네임스페이스
2. **커뮤니티 평가**: GitHub 스타, 다운로드 수, 업데이트 빈도
3. **호환성 확인**: Vite 버전, Node.js 버전 지원
4. **문서 품질**: README, TypeScript 지원, 예제 코드

## 참고 자료

- [Vite 플러그인 개발 가이드](https://vitejs.dev/guide/api-plugin.html)
- [Rollup 플러그인 호환성](https://vite.dev/guide/api-plugin.html#rollup-plugin-compatibility)
- [Awesome Vite 플러그인 목록](https://github.com/vitejs/awesome-vite)
- [플러그인 API 레퍼런스](https://vitejs.dev/guide/api-plugin.html)

## 관련 문서

- [../vite.md](./vite.md) - Vite 기본 설정
- [../vite-dev.md](./vite-dev.md) - 개발서버 설정
- [../vite-lib.md](./vite-lib.md) - 라이브러리 빌드 설정
