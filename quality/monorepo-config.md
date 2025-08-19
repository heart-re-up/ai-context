# 모노레포를 위한 설정 패키지 구현

모노레포에서 설정 파일을 패키지로 관리하는 실제 구현 방법을 다룹니다.

> 💡 **전체 구조와 전략**은 [`monorepo/config.md`](../monorepo/config.md)를 먼저 확인하세요.

## ESLint 설정 패키지 구현

### config/eslint-config/package.json

```json
{
  "name": "@project/eslint-config",
  "version": "1.0.0",
  "description": "Shared ESLint configuration for the project",
  "main": "base.mjs",
  "exports": {
    ".": "./base.mjs",
    "./react": "./react.mjs",
    "./node": "./node.mjs",
    "./lib": "./lib.mjs"
  },
  "files": ["*.mjs", "*.js"],
  "dependencies": {
    "@eslint/js": "^9.0.0",
    "@typescript-eslint/eslint-plugin": "^7.0.0",
    "@typescript-eslint/parser": "^7.0.0",
    "eslint-plugin-react": "^7.34.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.0",
    "globals": "^15.0.0",
    "typescript-eslint": "^7.0.0"
  },
  "peerDependencies": {
    "eslint": "^9.0.0",
    "typescript": "^5.0.0"
  }
}
```

### config/eslint-config/base.mjs

```javascript
import js from "@eslint/js";
import tseslint from "typescript-eslint";

export default tseslint.config(
  {
    ignores: [
      "**/dist/**",
      "**/node_modules/**",
      "**/.turbo/**",
      "**/*.tsbuildinfo",
      "**/coverage/**",
    ],
  },
  {
    extends: [js.configs.recommended, ...tseslint.configs.recommended],
    files: ["**/*.{ts,tsx,js,jsx}"],
    languageOptions: {
      ecmaVersion: 2020,
    },
    rules: {
      // 기본 TypeScript 규칙
      "@typescript-eslint/no-unused-vars": [
        "error",
        {
          argsIgnorePattern: "^_",
          varsIgnorePattern: "^_",
          caughtErrorsIgnorePattern: "^_",
        },
      ],
      "@typescript-eslint/no-explicit-any": "warn",
      "@typescript-eslint/prefer-nullish-coalescing": "error",
      "@typescript-eslint/prefer-optional-chain": "error",

      // 일반 JavaScript 규칙
      "no-console": ["warn", { allow: ["warn", "error"] }],
      "no-debugger": "error",
      "prefer-const": "error",
      "no-var": "error",
    },
  }
);
```

### config/eslint-config/react.mjs

```javascript
import baseConfig from "./base.mjs";
import react from "eslint-plugin-react";
import reactHooks from "eslint-plugin-react-hooks";
import reactRefresh from "eslint-plugin-react-refresh";
import globals from "globals";

export default [
  ...baseConfig,
  {
    files: ["**/*.{ts,tsx,jsx}"],
    languageOptions: {
      globals: globals.browser,
    },
    plugins: {
      react,
      "react-hooks": reactHooks,
      "react-refresh": reactRefresh,
    },
    rules: {
      // React Hooks 규칙
      ...reactHooks.configs.recommended.rules,

      // React Refresh 규칙 (HMR 최적화)
      "react-refresh/only-export-components": [
        "warn",
        { allowConstantExport: true },
      ],

      // React 규칙
      "react/react-in-jsx-scope": "off", // React 19+
      "react/jsx-boolean-value": ["error", "never"],
      "react/self-closing-comp": "error",
      "react/jsx-fragments": ["error", "syntax"],
      "react/no-array-index-key": "warn",
      "react/display-name": "error",
    },
    settings: {
      react: {
        version: "19.1",
      },
    },
  },
];
```

### config/eslint-config/lib.mjs

```javascript
import baseConfig from "./base.mjs";

export default [
  ...baseConfig,
  {
    files: ["**/*.{ts,tsx}"],
    rules: {
      // 라이브러리는 더 엄격한 규칙 적용
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/explicit-function-return-type": "warn",
      "no-console": "error", // 라이브러리에서는 console 금지
    },
  },
];
```

### config/eslint-config/node.mjs

```javascript
import baseConfig from "./base.mjs";
import globals from "globals";

export default [
  ...baseConfig,
  {
    files: ["**/*.{ts,js}"],
    languageOptions: {
      globals: globals.node,
    },
    rules: {
      "no-console": "off", // Node.js에서는 console 허용
      "@typescript-eslint/no-unused-vars": [
        "error",
        { argsIgnorePattern: "^(req|res|next)$" },
      ],
    },
  },
];
```

## TypeScript 설정 패키지 구현

### config/typescript-config/package.json

```json
{
  "name": "@project/typescript-config",
  "version": "1.0.0",
  "description": "Shared TypeScript configurations",
  "main": "base.json",
  "exports": {
    "./base": "./base.json",
    "./app": "./app.json",
    "./lib": "./lib.json",
    "./node": "./node.json"
  },
  "files": ["*.json"]
}
```

### config/typescript-config/base.json

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020"],
    "module": "ESNext",
    "skipLibCheck": true,
    "allowJs": true,

    /* 모듈 해결 */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,

    /* 타입 검사 */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,

    /* 기본 경로 */
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "exclude": ["node_modules", "dist", "**/*.test.*"]
}
```

### config/typescript-config/app.json

```json
{
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "noEmit": true,
    "jsx": "react-jsx",

    /* 앱 전용 경로 매핑 */
    "paths": {
      "@/*": ["src/*"],
      "@assets/*": ["src/assets/*"],
      "@components/*": ["src/components/*"],
      "@pages/*": ["src/pages/*"],
      "@utils/*": ["src/utils/*"]
    }
  },
  "include": ["vite-env.d.ts", "src/**/*.ts", "src/**/*.tsx"]
}
```

### config/typescript-config/lib.json

```json
{
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["ES2020", "DOM"],
    "declaration": true,
    "declarationMap": true,
    "outDir": "dist",
    "rootDir": "src",

    /* 라이브러리 빌드 설정 */
    "emitDeclarationOnly": false,
    "preserveSymlinks": true
  },
  "include": ["src/**/*.ts", "src/**/*.tsx"]
}
```

### config/typescript-config/node.json

```json
{
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["ES2020"],
    "module": "CommonJS",
    "moduleResolution": "node",
    "target": "ES2020",
    "outDir": "dist",
    "declaration": true,

    /* Node.js 환경 */
    "types": ["node"],
    "allowSyntheticDefaultImports": true,
    "esModuleInterop": true
  },
  "include": ["src/**/*.ts"],
  "exclude": ["**/*.test.ts", "**/*.spec.ts"]
}
```

## Prettier 설정 패키지 구현

### config/prettier-config/package.json

```json
{
  "name": "@project/prettier-config",
  "version": "1.0.0",
  "description": "Shared Prettier configuration",
  "main": "index.js",
  "files": ["index.js"]
}
```

### config/prettier-config/index.js

```javascript
/** @type {import("prettier").Config} */
module.exports = {
  // 기본 포매팅
  printWidth: 100,
  tabWidth: 2,
  useTabs: false,
  semi: true,
  singleQuote: true, // 작은따옴표 사용
  quoteProps: "as-needed",

  // JSX 설정
  jsxSingleQuote: false, // JSX는 큰따옴표

  // 기타 설정
  trailingComma: "all", // 모든 곳에 후행 쉼표
  bracketSpacing: true,
  bracketSameLine: false,
  arrowParens: "always",
  endOfLine: "lf",

  // 파일별 Override
  overrides: [
    {
      files: "*.md",
      options: {
        printWidth: 100,
        proseWrap: "always",
      },
    },
    {
      files: "*.json",
      options: {
        printWidth: 100,
      },
    },
  ],
};
```

## Vite 설정 패키지 구현

### config/vite-config/package.json

```json
{
  "name": "@project/vite-config",
  "version": "1.0.0",
  "description": "Shared Vite configurations",
  "main": "base.js",
  "exports": {
    "./app": "./app.js",
    "./lib": "./lib.js"
  },
  "files": ["*.js"],
  "dependencies": {
    "@vitejs/plugin-react": "^4.5.1",
    "vite-plugin-dts": "^3.9.1"
  },
  "peerDependencies": {
    "vite": "^6.0.0"
  }
}
```

### config/vite-config/app.js

```javascript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig(({ mode }) => ({
  plugins: [react()],
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version),
  },
  build: {
    outDir: "dist",
    sourcemap: mode !== "production",
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["react", "react-dom"],
          router: ["react-router"],
        },
      },
    },
  },
  server: {
    port: 3000,
    host: true,
  },
  preview: {
    port: 4173,
    host: true,
  },
}));
```

### config/vite-config/lib.js

```javascript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import dts from "vite-plugin-dts";
import { resolve } from "path";

export default defineConfig({
  plugins: [
    react(),
    dts({
      include: ["src"],
      exclude: ["src/**/*.test.*", "src/**/*.stories.*"],
    }),
  ],
  build: {
    lib: {
      entry: {
        index: resolve(process.cwd(), "src/index.ts"),
      },
      formats: ["es", "cjs"],
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

## 사용 예시

### 애플리케이션에서 사용

```javascript
// apps/web/eslint.config.mjs
import reactConfig from "@project/eslint-config/react";

export default reactConfig;
```

```javascript
// apps/web/vite.config.ts
import { mergeConfig } from "vite";
import appConfig from "@project/vite-config/app";

export default mergeConfig(appConfig, {
  // 프로젝트별 추가 설정
  define: {
    __APP_NAME__: '"Web App"',
  },
});
```

### 라이브러리에서 사용

```javascript
// packages/ui-components/eslint.config.mjs
import libConfig from "@project/eslint-config/lib";
import reactConfig from "@project/eslint-config/react";

export default [...libConfig, ...reactConfig];
```

```javascript
// packages/ui-components/vite.lib.config.ts
import { mergeConfig } from "vite";
import libConfig from "@project/vite-config/lib";

export default mergeConfig(libConfig, {
  build: {
    lib: {
      entry: {
        index: resolve(__dirname, "src/index.ts"),
        components: resolve(__dirname, "src/components/index.ts"),
        hooks: resolve(__dirname, "src/hooks/index.ts"),
      },
    },
  },
});
```

## 설정 패키지 관리 팁

### 1. 버전 관리

```bash
# 설정 변경 후 버전 업데이트
cd config/eslint-config
npm version patch

# 또는 changeset 사용
pnpm changeset
```

### 2. 의존성 업데이트

```bash
# 모든 모듈에서 설정 패키지 업데이트
pnpm update @project/eslint-config --recursive
```

### 3. 설정 검증

```bash
# 모든 모듈에서 설정 테스트
pnpm -r lint
pnpm -r format
pnpm -r typecheck
```

### 4. 문제 해결

```bash
# 설정 패키지 캐시 클리어
rm -rf node_modules/.cache
pnpm install

# 특정 모듈의 설정 확인
cd apps/web
pnpm eslint --print-config src/index.ts
```

이렇게 설정 패키지를 구현하면 모노레포 전체에서 일관된 코드 품질을 유지할 수 있습니다.
