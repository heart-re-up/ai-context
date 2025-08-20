# ESLint 설정 가이드

ESLint는 JavaScript/TypeScript 코드의 문제를 찾고 코딩 스타일을 일관성 있게 유지하는 린팅 도구입니다.

>  **모노레포에서 ESLint 설정 패키지 구현**은 [monorepo-config.md](./monorepo-config.md#eslint-설정-패키지-구현)를 참고하세요.

## ESLint 9+ Flat Config 소개

ESLint 9부터는 새로운 "Flat Config" 시스템을 사용합니다. 기존의 `.eslintrc.*` 파일을 대체하는 `eslint.config.mjs` 파일을 사용합니다.

### 기존 vs 새로운 설정 방식

```javascript
// 기존 방식 (.eslintrc.js) - 더 이상 권장하지 않음
module.exports = {
  extends: ["@typescript-eslint/recommended"],
  plugins: ["react"],
  rules: {
    "no-console": "warn",
  },
};

// 새로운 방식 (eslint.config.mjs) - ESLint 9+
export default [
  {
    files: ["**/*.{js,ts,jsx,tsx}"],
    plugins: {
      "@typescript-eslint": typescriptEslint,
    },
    rules: {
      "no-console": "warn",
    },
  },
];
```

### Flat Config의 장점

1. **더 명확한 구성**: 설정이 배열 형태로 명시적
2. **더 나은 성능**: 설정 해석이 더 빠름
3. **더 유연한 파일 매칭**: glob 패턴을 더 정확히 제어
4. **플러그인 로딩 개선**: 플러그인을 명시적으로 import

## 1. 기본 ESLint 설정

### 가장 기본적인 설정

```javascript
// eslint.config.mjs
import js from "@eslint/js";

export default [
  // 기본 JavaScript 권장 설정
  js.configs.recommended,

  // 기본 설정
  {
    files: ["**/*.{js,jsx}"],
    languageOptions: {
      ecmaVersion: 2022,
      sourceType: "module",
    },
    rules: {
      // 기본 규칙
      "no-console": "warn",
      "no-debugger": "error",
      "prefer-const": "error",
      "no-var": "error",
    },
  },
];
```

### 환경별 글로벌 변수 설정

```javascript
import js from "@eslint/js";
import globals from "globals";

export default [
  js.configs.recommended,
  {
    files: ["**/*.js"],
    languageOptions: {
      globals: {
        ...globals.browser, // 브라우저 환경
        ...globals.node, // Node.js 환경
        // 커스텀 글로벌 변수
        myGlobalVar: "readonly",
      },
    },
  },
];
```

## 2. TypeScript 통합

TypeScript를 사용하는 프로젝트에서는 `typescript-eslint`를 추가합니다.

### 기본 TypeScript 설정

```javascript
// eslint.config.mjs
import js from "@eslint/js";
import tseslint from "typescript-eslint";

export default tseslint.config(
  // JavaScript 기본 규칙
  js.configs.recommended,

  // TypeScript 권장 규칙
  ...tseslint.configs.recommended,

  {
    files: ["**/*.{ts,tsx}"],
    languageOptions: {
      ecmaVersion: 2022,
      sourceType: "module",
    },
    rules: {
      // TypeScript 특화 규칙
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
    },
  }
);
```

### 타입 정보를 활용한 고급 TypeScript 설정

```javascript
import tseslint from "typescript-eslint";

export default tseslint.config(
  // ... 기본 설정
  {
    files: ["**/*.{ts,tsx}"],
    languageOptions: {
      parserOptions: {
        project: true, // tsconfig.json 사용
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      // 타입 정보가 필요한 규칙들
      "@typescript-eslint/no-floating-promises": "error",
      "@typescript-eslint/no-misused-promises": "error",
      "@typescript-eslint/require-await": "error",
      "@typescript-eslint/await-thenable": "error",
      "@typescript-eslint/no-unnecessary-type-assertion": "error",
    },
  }
);
```

## 3. React 프로젝트 설정

React 프로젝트에서는 React 전용 규칙들을 추가합니다.

### React 기본 설정

```javascript
// eslint.config.mjs
import js from "@eslint/js";
import tseslint from "typescript-eslint";
import react from "eslint-plugin-react";
import reactHooks from "eslint-plugin-react-hooks";
import globals from "globals";

export default tseslint.config(
  js.configs.recommended,
  ...tseslint.configs.recommended,

  {
    files: ["**/*.{js,jsx,ts,tsx}"],
    languageOptions: {
      ecmaVersion: 2022,
      globals: globals.browser,
    },
    plugins: {
      react,
      "react-hooks": reactHooks,
    },
    rules: {
      // React 기본 규칙
      ...react.configs.recommended.rules,
      ...reactHooks.configs.recommended.rules,

      // React 19+ 자동 JSX 런타임
      "react/react-in-jsx-scope": "off",

      // React 코딩 규칙
      "react/jsx-boolean-value": ["error", "never"],
      "react/self-closing-comp": "error",
      "react/jsx-fragments": ["error", "syntax"],
      "react/no-array-index-key": "warn",
      "react/display-name": "error",
    },
    settings: {
      react: {
        version: "detect", // 자동 감지 또는 "19.1"
      },
    },
  }
);
```

### React with HMR (Hot Module Replacement) 설정

```javascript
import reactRefresh from "eslint-plugin-react-refresh";

export default tseslint.config(
  // ... 기본 설정
  {
    files: ["**/*.{jsx,tsx}"],
    plugins: {
      // ... 기존 플러그인
      "react-refresh": reactRefresh,
    },
    rules: {
      // HMR 최적화를 위한 규칙
      "react-refresh/only-export-components": [
        "warn",
        { allowConstantExport: true },
      ],
    },
  }
);
```

## 4. 파일 패턴별 설정

### 파일별 다른 규칙 적용

```javascript
export default [
  // TypeScript 파일
  {
    files: ["**/*.{ts,tsx}"],
    rules: {
      "@typescript-eslint/no-explicit-any": "error",
    },
  },

  // JavaScript 파일 (더 관대한 규칙)
  {
    files: ["**/*.{js,jsx}"],
    rules: {
      "no-undef": "error", // JS에서는 undef 체크 필요
    },
  },

  // 테스트 파일
  {
    files: ["**/*.test.{js,ts,jsx,tsx}", "**/__tests__/**/*"],
    rules: {
      "no-console": "off", // 테스트에서는 console 허용
      "@typescript-eslint/no-explicit-any": "off",
    },
  },

  // 설정 파일
  {
    files: ["*.config.{js,ts}", "*.config.*.{js,ts}"],
    languageOptions: {
      globals: {
        ...globals.node, // Node.js 환경
      },
    },
    rules: {
      "no-console": "off",
    },
  },
];
```

### 무시할 파일 패턴

```javascript
export default [
  // 무시할 파일들
  {
    ignores: [
      "**/dist/**",
      "**/build/**",
      "**/node_modules/**",
      "**/.turbo/**",
      "**/*.tsbuildinfo",
      "**/coverage/**",
      "**/*.min.js",
      "**/generated/**", // 자동 생성 파일
    ],
  },

  // ... 나머지 설정
];
```

## 5. Import 정렬 및 관리

### Import 정렬 자동화

```bash
# 의존성 설치
pnpm add -D eslint-plugin-simple-import-sort
```

```javascript
import simpleImportSort from "eslint-plugin-simple-import-sort";

export default [
  {
    plugins: {
      "simple-import-sort": simpleImportSort,
    },
    rules: {
      "simple-import-sort/imports": "error",
      "simple-import-sort/exports": "error",
    },
  },
];
```

### Import 그룹 커스터마이징

```javascript
export default [
  {
    plugins: {
      "simple-import-sort": simpleImportSort,
    },
    rules: {
      "simple-import-sort/imports": [
        "error",
        {
          groups: [
            // React 관련
            ["^react", "^@?\\w"],
            // 내부 패키지
            ["^@project"],
            // 상대 경로
            [
              "^\\.\\.(?!/?$)",
              "^\\.\\./?$",
              "^\\./(?=.*/)(?!/?$)",
              "^\\.(?!/?$)",
              "^\\./?$",
            ],
            // 스타일
            ["^.+\\.s?css$"],
          ],
        },
      ],
    },
  },
];
```

## 6. 모노레포 설정 전략

### 설정 패키지로 관리 (권장)

모노레포에서는 ESLint 설정을 패키지로 만들어 중앙에서 관리하는 것이 효율적입니다.

#### config/eslint-config/package.json

```json
{
  "name": "@project/eslint-config",
  "version": "1.0.0",
  "description": "Shared ESLint configuration",
  "main": "base.mjs",
  "exports": {
    ".": "./base.mjs",
    "./react": "./react.mjs",
    "./node": "./node.mjs",
    "./lib": "./lib.mjs"
  },
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

#### config/eslint-config/base.mjs

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
  js.configs.recommended,
  ...tseslint.configs.recommended,
  {
    files: ["**/*.{ts,tsx,js,jsx}"],
    languageOptions: {
      ecmaVersion: 2020,
    },
    rules: {
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
      "no-console": ["warn", { allow: ["warn", "error"] }],
      "no-debugger": "error",
      "prefer-const": "error",
      "no-var": "error",
    },
  }
);
```

#### config/eslint-config/react.mjs

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
      ...reactHooks.configs.recommended.rules,
      "react-refresh/only-export-components": [
        "warn",
        { allowConstantExport: true },
      ],
      "react/react-in-jsx-scope": "off",
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

#### config/eslint-config/lib.mjs

```javascript
import baseConfig from "./base.mjs";

export default [
  ...baseConfig,
  {
    files: ["**/*.{ts,tsx}"],
    rules: {
      // 라이브러리는 더 엄격한 규칙
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/explicit-function-return-type": "warn",
      "no-console": "error",
    },
  },
];
```

#### config/eslint-config/node.mjs

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
      "no-console": "off",
      "@typescript-eslint/no-unused-vars": [
        "error",
        { argsIgnorePattern: "^(req|res|next)$" },
      ],
    },
  },
];
```

### 사용 방법

#### 애플리케이션에서 사용

```javascript
// apps/web/eslint.config.mjs
import reactConfig from "@project/eslint-config/react";

export default reactConfig;
```

#### 라이브러리에서 사용

```javascript
// packages/ui-components/eslint.config.mjs
import libConfig from "@project/eslint-config/lib";
import reactConfig from "@project/eslint-config/react";

export default [...libConfig, ...reactConfig];
```

#### 설정 확장 및 재정의

```javascript
// apps/admin/eslint.config.mjs
import reactConfig from "@project/eslint-config/react";

export default [
  ...reactConfig,
  {
    rules: {
      "no-console": "off", // 관리자 앱에서는 console 허용
    },
  },
];
```

## 7. 성능 최적화

### 캐시 활성화

```json
// package.json
{
  "scripts": {
    "lint": "eslint . --cache --cache-location .eslintcache",
    "lint:fix": "eslint . --cache --fix",
    "lint:ci": "eslint . --max-warnings 0"
  }
}
```

### 병렬 처리

```bash
# 모노레포에서 병렬 린트
pnpm -r --parallel lint
```

## 8. IDE 통합

### VS Code 설정

```json
// .vscode/settings.json
{
  "eslint.enable": true,
  "eslint.validate": [
    "javascript",
    "typescript",
    "javascriptreact",
    "typescriptreact"
  ],
  "eslint.format.enable": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit"
  },
  "eslint.workingDirectories": [{ "mode": "auto" }]
}
```

## 9. 환경별 설정 패턴

### 개발/프로덕션 환경 분리

```javascript
export default [
  {
    files: ["**/*.{js,ts,jsx,tsx}"],
    rules: {
      // 개발 환경에서만 경고
      "no-console": process.env.NODE_ENV === "production" ? "error" : "warn",
      "no-debugger": process.env.NODE_ENV === "production" ? "error" : "warn",
    },
  },
];
```

### 라이브러리 vs 애플리케이션

```javascript
// 라이브러리용 - 더 엄격
export const libConfig = [
  {
    files: ["**/*.{ts,tsx}"],
    rules: {
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/explicit-function-return-type": "warn",
      "no-console": "error", // 라이브러리에서는 console 금지
    },
  },
];

// 애플리케이션용 - 더 관대
export const appConfig = [
  {
    files: ["**/*.{ts,tsx}"],
    rules: {
      "@typescript-eslint/no-explicit-any": "warn",
      "no-console": ["warn", { allow: ["warn", "error"] }],
    },
  },
];
```

## 문제 해결

### 자주 발생하는 문제

1. **설정 파일을 찾지 못함**

   ```bash
   eslint --print-config src/index.ts
   ```

2. **특정 파일이 무시됨**

   ```bash
   eslint --debug src/problematic-file.ts
   ```

3. **성능 이슈**
   ```bash
   TIMING=1 eslint src/
   ```

### 마이그레이션 가이드

#### .eslintrc에서 flat config로 이전

```javascript
// 기존 .eslintrc.js
module.exports = {
  extends: ["@typescript-eslint/recommended", "plugin:react/recommended"],
  plugins: ["react-hooks"],
  rules: {
    "no-console": "warn",
  },
};

// 새로운 eslint.config.mjs
import tseslint from "typescript-eslint";
import react from "eslint-plugin-react";
import reactHooks from "eslint-plugin-react-hooks";

export default tseslint.config(...tseslint.configs.recommended, {
  plugins: {
    react,
    "react-hooks": reactHooks,
  },
  rules: {
    ...react.configs.recommended.rules,
    ...reactHooks.configs.recommended.rules,
    "no-console": "warn",
  },
});
```

이제 ESLint 설정이 기본부터 고급까지 체계적으로 구성되었습니다. 다음으로 다른 파일들도 정리하겠습니다.
