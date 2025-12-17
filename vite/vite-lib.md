# Vite 라이브러리 빌드 가이드

Vite를 사용하여 모듈을 패키징하고 라이브러리로 배포하는 방법에 대한 가이드입니다.

## 기본 라이브러리 설정

### 기본 구성

```typescript
// vite.lib.config.ts
import { readFileSync } from "fs";
import { resolve } from "path";
import { defineConfig } from "vite";
import dts from "vite-plugin-dts";

// package.json 읽기
const pkg = JSON.parse(readFileSync("./package.json", "utf-8"));

export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, "src/index.ts"),
      name: "MyLibrary", // UMD 빌드 시 전역 변수명
      fileName: "index", // 출력 파일명
      formats: ["es", "cjs"], // 출력 형식
    },
    outDir: "dist",
    sourcemap: true,
    rollupOptions: {
      external: ["react", "react-dom"],
      output: {
        globals: {
          react: "React",
          "react-dom": "ReactDOM",
        },
      },
    },
  },
});
```

### TypeScript 선언 파일 생성

```typescript
import dts from "vite-plugin-dts";

export default defineConfig({
  plugins: [
    dts({
      rollupTypes: true, // 단일 .d.ts 파일로 병합
      tsconfigPath: "./tsconfig.build.json",
      include: ["src/**/*"],
      exclude: ["src/**/*.test.ts", "src/**/*.test.tsx"],
      insertTypesEntry: true, // package.json types 필드 자동 설정
    }),
  ],
});
```

## 다중 진입점 라이브러리

### 여러 진입점 설정

```typescript
export default defineConfig({
  build: {
    lib: {
      entry: {
        index: resolve(__dirname, "src/index.ts"),
        hooks: resolve(__dirname, "src/hooks/index.ts"),
        utils: resolve(__dirname, "src/utils/index.ts"),
        components: resolve(__dirname, "src/components/index.ts"),
      },
      formats: ["es", "cjs"],
      fileName: (format, entryName) =>
        `${entryName}.${format === "es" ? "js" : "cjs"}`,
    },
  },
});
```

### package.json 설정

```json
{
  "name": "@myorg/my-library",
  "version": "1.0.0",
  "type": "module",
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "require": "./dist/index.cjs",
      "types": "./dist/index.d.ts"
    },
    "./hooks": {
      "import": "./dist/hooks.js",
      "require": "./dist/hooks.cjs",
      "types": "./dist/hooks.d.ts"
    },
    "./utils": {
      "import": "./dist/utils.js",
      "require": "./dist/utils.cjs",
      "types": "./dist/utils.d.ts"
    },
    "./components": {
      "import": "./dist/components.js",
      "require": "./dist/components.cjs",
      "types": "./dist/components.d.ts"
    }
  },
  "files": ["dist"],
  "sideEffects": false
}
```

### 사용 예시

라이브러리 사용측에서는 다음과 같이 각 진입점별로 모듈을 import할 수 있습니다:

```typescript
// 1. 기본 진입점에서 import
import { Button, Input, Card } from "@myorg/my-library";

// 2. hooks만 별도로 import (코드 분할)
import { useLocalStorage, useDebounce } from "@myorg/my-library/hooks";

// 3. utils만 별도로 import (코드 분할)
import { formatDate, debounce, deepClone } from "@myorg/my-library/utils";

// 4. 특정 컴포넌트만 import (코드 분할)
import { Button, Modal } from "@myorg/my-library/components";

// 5. 믹스 사용 - 필요한 것만 개별적으로 import
import { Card } from "@myorg/my-library";
import { useLocalStorage } from "@myorg/my-library/hooks";
import { formatDate } from "@myorg/my-library/utils";

function MyComponent() {
  const [data, setData] = useLocalStorage("myData", {});

  return (
    <Card>
      <p>{formatDate(new Date())}</p>
      <Button onClick={() => setData({ updated: true })}>Update</Button>
    </Card>
  );
}
```

**Tree-shaking 효과:**

```typescript
//  전체 라이브러리 번들 (모든 코드가 포함됨)
import { useLocalStorage } from "@myorg/my-library";

//  hooks만 번들에 포함 (더 작은 번들 크기)
import { useLocalStorage } from "@myorg/my-library/hooks";
```

## React 컴포넌트 라이브러리

### 공통 컴포넌트 라이브러리 설정

```typescript
import react from "@vitejs/plugin-react";
import { libInjectCss } from "vite-plugin-lib-inject-css";

export default defineConfig({
  plugins: [
    react(),
    libInjectCss(), // CSS를 JS에 인라인으로 주입
    dts({
      rollupTypes: true,
      tsconfigPath: "./tsconfig.json",
    }),
  ],

  build: {
    lib: {
      entry: resolve(__dirname, "src/index.ts"),
      formats: ["es", "cjs"],
      fileName: (format) => `index.${format === "es" ? "js" : "cjs"}`,
    },
    rollupOptions: {
      external: [
        "react",
        "react-dom",
        "react/jsx-runtime",
        // peer dependencies
        "@mui/material",
        "@emotion/react",
        "@emotion/styled",
      ],
      output: {
        globals: {
          react: "React",
          "react-dom": "ReactDOM",
        },
      },
    },
  },
});
```

### CSS 처리 옵션

#### 1. CSS 별도 파일로 추출

```typescript
export default defineConfig({
  build: {
    cssCodeSplit: true, // CSS 분리
    lib: {
      // ...
    },
    rollupOptions: {
      output: {
        assetFileNames: "assets/[name][extname]",
      },
    },
  },
});
```

#### 2. CSS-in-JS 라이브러리 (추천)

```typescript
// 사용자가 CSS를 별도로 import하지 않아도 됨
import { libInjectCss } from "vite-plugin-lib-inject-css";

export default defineConfig({
  plugins: [react(), libInjectCss()],
});
```

## 외부 의존성 관리

### peerDependencies 자동 처리

```typescript
import { readFileSync } from "fs";

const pkg = JSON.parse(readFileSync("./package.json", "utf-8"));

export default defineConfig({
  build: {
    rollupOptions: {
      external: [
        // React 관련
        "react",
        "react-dom",
        "react/jsx-runtime",
        // peerDependencies 자동 추가
        ...Object.keys(pkg.peerDependencies || {}),
        // 선택적으로 dependencies도 제외
        ...Object.keys(pkg.dependencies || {}),
      ],
    },
  },
});
```

### 특정 의존성 번들에 포함

```typescript
export default defineConfig({
  build: {
    rollupOptions: {
      external: (id) => {
        // React는 제외하고 lodash는 번들에 포함
        if (id.includes("react")) return true;
        if (id.includes("lodash")) return false;
        return false;
      },
    },
  },
});
```

## 고급 빌드 설정

### Tree-shaking 최적화

#### Vite Entry 설정 - 개별 번들 생성

```typescript
export default defineConfig({
  build: {
    lib: {
      entry: {
        // 각 기능을 개별 번들로 분리
        index: resolve(__dirname, "src/index.ts"),
        "components/button": resolve(
          __dirname,
          "src/components/Button/index.ts"
        ),
        "components/input": resolve(__dirname, "src/components/Input/index.ts"),
        "hooks/useLocalStorage": resolve(
          __dirname,
          "src/hooks/useLocalStorage.ts"
        ),
      },
      formats: ["es", "cjs"], // 필요에 따라 선택 (es만 또는 둘 다)
    },
    rollupOptions: {
      output: {
        preserveModules: true, // 소스 디렉토리 구조 그대로 유지
        preserveModulesRoot: "src",
      },
    },
  },
});
```

**핵심 개념:**

- **개별 entry**: 각 파일이 독립적인 번들로 생성됨
- **사용자 import 시**: 필요한 번들만 로드되어 **번들 크기 최소화**
- **preserveModules**: 소스의 디렉토리 구조를 빌드 결과에서도 동일하게 유지
- **다중 포맷 출력**: 프로젝트 요구사항에 맞게 다양한 모듈 포맷 선택 가능

**빌드 결과:**

```
dist/
  index.js, index.cjs         (5KB each)
  components/
    button.js, button.cjs     (3KB each)
    input.js, input.cjs       (4KB each)
  hooks/
    useLocalStorage.js, useLocalStorage.cjs  (2KB each)
```

#### Package.json Exports - 구조 기반 접근

```json
{
  "sideEffects": false,
  "exports": {
    ".": {
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./components/*": {
      "import": "./dist/components/*.js",
      "require": "./dist/components/*.cjs"
    },
    "./hooks/*": {
      "import": "./dist/hooks/*.js",
      "require": "./dist/hooks/*.cjs"
    }
  }
}
```

**Subpath 패턴의 정확한 매핑:**

- `./components/*` 패턴에서 `*`는 **정확히 하나의 파일명**과 매칭
- 글로브 패턴과 달리 **1:1 문자 교체** 방식으로 동작

**매핑 예시:**

```typescript
// 사용자 import
import { Button } from '@mylib/components/button';

// 내부 변환 과정
'./components/*' 패턴 → '*' 부분이 'button'으로 교체
→ './dist/components/button.js' 경로로 해석
```

#### 배럴 Export와의 차이점

** 배럴 방식 (전통적)**

```typescript
// src/components/index.ts (배럴 파일)
export { Button } from "./Button";
export { Input } from "./Input";
export { Modal } from "./Modal";
// ... 모든 컴포넌트들

// 사용자 측
import { Button } from "@mylib/components"; // 모든 컴포넌트가 번들에 포함될 위험
```

** 구조 기반 방식 (Tree-shaking 최적화)**

```typescript
// 배럴 파일 없음 - 개별 파일로 직접 접근

// 사용자 측
import { Button } from "@mylib/components/button"; // Button만 번들에 포함 (3KB)
```

#### 사용 예시와 번들 크기 비교

```typescript
// 개별 import - 최적화된 방식
import { Button } from "@mylib/components/button"; // 3KB
import { useLocalStorage } from "@mylib/hooks/useLocalStorage"; // 2KB
// 총 번들 크기: 5KB

// 배럴 import - 비효율적일 수 있는 방식
import { Button } from "@mylib/components"; // 3KB + 다른 컴포넌트들
import { useLocalStorage } from "@mylib/hooks"; // 2KB + 다른 훅들
// 총 번들 크기: 15KB+ (tree-shaking 실패 시)
```

#### 지원 가능한 모듈 포맷

**Vite에서 지원하는 모든 포맷:**

- **`es`** (ES Modules)

  - **문법**: `import`/`export`
  - **환경**: 모던 브라우저, Node.js v14+, 모든 번들러
  - **특징**: Tree-shaking 지원, 정적 분석 가능
  - **파일**: `.js` (ES 모듈)

- **`cjs`** (CommonJS)

  - **문법**: `require()`/`module.exports`
  - **환경**: Node.js, 일부 번들러
  - **특징**: 동적 로딩, 런타임 해석
  - **파일**: `.cjs` 또는 `.js` (CommonJS)

- **`umd`** (Universal Module Definition)

  - **문법**: 환경에 따라 자동 감지
  - **환경**: 브라우저(전역), AMD, CommonJS, ES 모듈
  - **특징**: 범용 호환성, CDN 친화적
  - **파일**: `.umd.js`

- **`iife`** (Immediately Invoked Function Expression)
  - **문법**: 전역 변수
  - **환경**: 브라우저 전용 (`<script>` 태그)
  - **특징**: 즉시 실행, 네임스페이스 오염 최소화
  - **파일**: `.iife.js`

#### 유즈케이스별 포맷 선택 가이드

#####  케이스 1: Node.js 환경 전용 React 라이브러리

```typescript
// SSR, 테스트, 빌드 도구에서만 사용
export default defineConfig({
  build: {
    lib: {
      formats: ["cjs"], // CommonJS만
      fileName: "index",
    },
  },
});
```

**package.json:**

```json
{
  "main": "./dist/index.cjs",
  "exports": {
    ".": {
      "require": "./dist/index.cjs"
    }
  }
}
```

**사용 환경:**

- Next.js SSR
- Jest 테스트 환경
- Node.js 백엔드에서 React 컴포넌트 렌더링

#####  케이스 2: Node.js + 브라우저 CDN 지원

```typescript
// 가장 범용적인 설정
export default defineConfig({
  build: {
    lib: {
      name: "MyReactLib", // 브라우저 전역 변수명
      formats: ["es", "cjs", "umd"],
      fileName: (format) => {
        switch (format) {
          case "es":
            return "index.esm.js";
          case "cjs":
            return "index.cjs";
          case "umd":
            return "index.umd.js";
          default:
            return "index.js";
        }
      },
    },
    rollupOptions: {
      external: ["react", "react-dom"],
      output: {
        globals: {
          react: "React",
          "react-dom": "ReactDOM",
        },
      },
    },
  },
});
```

**package.json:**

```json
{
  "main": "./dist/index.cjs", // Node.js 기본
  "module": "./dist/index.esm.js", // 번들러용
  "unpkg": "./dist/index.umd.js", // CDN용
  "exports": {
    ".": {
      "import": "./dist/index.esm.js",
      "require": "./dist/index.cjs",
      "browser": "./dist/index.umd.js"
    }
  }
}
```

**사용 환경:**

```html
<!-- CDN 사용 -->
<script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script src="https://unpkg.com/my-react-lib/dist/index.umd.js"></script>
<script>
  const { Button } = MyReactLib;
</script>
```

```typescript
// Node.js 사용
const { Button } = require("my-react-lib");

// 모던 번들러 사용
import { Button } from "my-react-lib";
```

##### 📱 케이스 3: 브라우저 전용 CDN 라이브러리

```typescript
// 경량화된 브라우저 전용
export default defineConfig({
  build: {
    lib: {
      name: "MyUIKit",
      formats: ["umd", "iife"], // 브라우저 전용
      fileName: (format) => `my-ui-kit.${format}.js`,
    },
    rollupOptions: {
      external: ["react", "react-dom"],
      output: {
        globals: {
          react: "React",
          "react-dom": "ReactDOM",
        },
      },
    },
  },
});
```

**package.json:**

```json
{
  "browser": "./dist/my-ui-kit.umd.js",
  "unpkg": "./dist/my-ui-kit.umd.js",
  "jsdelivr": "./dist/my-ui-kit.umd.js"
}
```

**사용 환경:**

```html
<!-- UMD 사용 (권장) -->
<script src="https://unpkg.com/my-ui-kit/dist/my-ui-kit.umd.js"></script>
<script>
  const { Button, Modal } = MyUIKit;
</script>

<!-- IIFE 사용 (더 가벼움) -->
<script src="https://unpkg.com/my-ui-kit/dist/my-ui-kit.iife.js"></script>
<script>
  const { Button, Modal } = MyUIKit;
</script>
```

#####  케이스 4: 모던 환경 최적화

```typescript
// 최신 환경만 지원 (권장)
export default defineConfig({
  build: {
    lib: {
      formats: ["es"], // ES 모듈만
      fileName: "index",
    },
  },
});
```

**package.json:**

```json
{
  "type": "module",
  "exports": {
    ".": "./dist/index.js"
  },
  "engines": {
    "node": ">=14"
  }
}
```

**장점:**

- 가장 작은 번들 크기
- 최적의 Tree-shaking
- 빠른 빌드 시간
- 미래 지향적

#### 포맷별 특징 요약

| 포맷     | 번들 크기 | Tree-shaking | 호환성 | CDN 지원 | 추천 용도              |
| -------- | --------- | ------------ | ------ | -------- | ---------------------- |
| **es**   |     |        |    |        | 모던 환경, 최적 성능   |
| **cjs**  |     |          |  |        | Node.js, 레거시 번들러 |
| **umd**  |       |            |  |    | 범용 호환, CDN 배포    |
| **iife** |       |            |      |      | 브라우저 전용, 단순함  |

**핵심 장점:**

1. **확실한 번들 최적화**: 사용자가 import한 것만 정확히 번들에 포함
2. **구조 유지**: 개발자에게 친숙한 디렉토리 구조 그대로 사용
3. **명확한 API**: 어떤 모듈이 사용 가능한지 패키지 구조로 바로 파악 가능
4. **유연한 배포**: 다양한 환경과 사용 사례에 맞는 포맷 선택 가능

### 번들 최적화

```typescript
export default defineConfig({
  build: {
    lib: {
      // ...
    },
    rollupOptions: {
      output: {
        // 청크 분할 비활성화 (라이브러리는 단일 파일 권장)
        manualChunks: undefined,
        inlineDynamicImports: true,
      },
    },
    // 최적화 설정
    minify: "esbuild", // 빠른 minify
    target: "es2015", // 타겟 브라우저 설정
    chunkSizeWarningLimit: 1000, // 청크 크기 경고 임계값
  },
});
```

## 빌드 분석 및 최적화

### 번들 분석기 통합

```typescript
import { visualizer } from "rollup-plugin-visualizer";

export default defineConfig({
  plugins: [
    visualizer({
      open: false,
      gzipSize: true,
      brotliSize: true,
      filename: "./dist/bundle-report.html",
    }),
  ],
});
```

### 빌드 성능 측정

```typescript
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    // 빌드 시간 측정
    rollupOptions: {
      plugins: [
        {
          name: "build-timer",
          buildStart() {
            this.startTime = Date.now();
          },
          generateBundle() {
            const duration = Date.now() - this.startTime;
            console.log(`Build completed in ${duration}ms`);
          },
        },
      ],
    },
  },
});
```

## 멀티 포맷 빌드

### 모든 포맷 지원

```typescript
export default defineConfig({
  build: {
    lib: {
      entry: resolve(__dirname, "src/index.ts"),
      name: "MyLibrary",
      formats: ["es", "cjs", "umd", "iife"],
      fileName: (format) => {
        switch (format) {
          case "es":
            return "index.esm.js";
          case "cjs":
            return "index.cjs.js";
          case "umd":
            return "index.umd.js";
          case "iife":
            return "index.iife.js";
          default:
            return "index.js";
        }
      },
    },
  },
});
```

### 브라우저/Node.js 호환성

```typescript
export default defineConfig({
  build: {
    rollupOptions: {
      output: [
        // 브라우저용 (ES 모듈)
        {
          format: "es",
          entryFileNames: "index.esm.js",
          dir: "dist/esm",
        },
        // Node.js용 (CommonJS)
        {
          format: "cjs",
          entryFileNames: "index.cjs.js",
          dir: "dist/cjs",
        },
      ],
    },
  },
});
```

## 개발 워크플로우

### Watch 모드 설정

```json
// package.json
{
  "scripts": {
    "build": "vite build",
    "build:watch": "vite build --watch",
    "build:lib": "vite build -c vite.lib.config.ts",
    "build:lib:watch": "vite build -c vite.lib.config.ts --watch",
    "prepublishOnly": "npm run build:lib"
  }
}
```

### 라이브러리 테스트 설정

라이브러리 테스트 환경 구성에 대한 자세한 내용은 [Vitest 테스트 설정 가이드](../vitest/README.md)를 참고하세요.

```bash
# 테스트 관련 스크립트 (package.json)
"scripts": {
  "test": "vitest",
  "test:coverage": "vitest run --coverage",
  "test:ui": "vitest --ui"
}
```

## 배포 설정

### npm 배포 준비

```json
// package.json
{
  "name": "@myorg/my-library",
  "version": "1.0.0",
  "description": "My awesome library",
  "keywords": ["react", "typescript", "vite"],
  "author": "Your Name",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/myorg/my-library.git"
  },
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "files": ["dist", "README.md"],
  "engines": {
    "node": ">=16"
  },
  "peerDependencies": {
    "react": ">=17.0.0",
    "react-dom": ">=17.0.0"
  }
}
```

### .npmignore 설정

```
# 소스 파일
src/
tsconfig.json
vite.config.ts
vite.lib.config.ts

# 개발 도구
.eslintrc.js
.prettierrc
jest.config.js

# 테스트
__tests__/
*.test.*
*.spec.*

# 문서
docs/
examples/
```

## 라이브러리 문서화

### API 문서 자동 생성

```typescript
// TypeDoc과 연동
import { defineConfig } from "vite";

export default defineConfig({
  plugins: [
    {
      name: "typedoc-generator",
      closeBundle() {
        // 빌드 완료 후 문서 생성
        exec("typedoc --out docs src/index.ts");
      },
    },
  ],
});
```

### README 예제

```markdown
# @myorg/my-library

React components and hooks library built with Vite.

## Installation

\`\`\`bash
npm install @myorg/my-library
\`\`\`

## Usage

\`\`\`tsx
import { Button, useLocalStorage } from '@myorg/my-library';

function App() {
const [count, setCount] = useLocalStorage('count', 0);

return (
<Button onClick={() => setCount(count + 1)}>
Count: {count}
</Button>
);
}
\`\`\`
```

## 문제 해결

### 자주 발생하는 이슈

1. **TypeScript 타입이 생성되지 않음**

   ```typescript
   // tsconfig.build.json 별도 생성
   {
     "extends": "./tsconfig.json",
     "compilerOptions": {
       "declaration": true,
       "emitDeclarationOnly": true,
       "outDir": "dist"
     },
     "include": ["src/**/*"],
     "exclude": ["**/*.test.*", "**/*.spec.*"]
   }
   ```

2. **외부 의존성이 번들에 포함됨**

   ```typescript
   // external 설정 확인
   rollupOptions: {
     external: ['react', 'react-dom'],
   }
   ```

3. **CSS가 작동하지 않음**

   ```typescript
   // libInjectCss 플러그인 사용
   import { libInjectCss } from "vite-plugin-lib-inject-css";
   ```

4. **Tree-shaking이 작동하지 않음**
   ```json
   // package.json
   {
     "sideEffects": false,
     "module": "./dist/index.js"
   }
   ```

## 모범 사례

1. **명확한 API**: 일관되고 직관적인 API 설계
2. **타입 안전성**: 완전한 TypeScript 지원
3. **번들 크기**: 최소한의 번들 크기 유지
4. **호환성**: 다양한 환경에서 동작 보장
5. **문서화**: 완전한 문서와 예제 제공
6. **테스트**: 충분한 테스트 커버리지
7. **버전 관리**: semantic versioning 준수

## 참고 자료

- [Vite 라이브러리 모드](https://vitejs.dev/guide/build.html#library-mode)
- [TypeScript 선언 파일](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html)
- [npm 패키지 배포](https://docs.npmjs.com/creating-and-publishing-unscoped-public-packages)
