# Packages 디렉토리 가이드

이 문서는 모노레포의 `packages/` 디렉토리에 대한 상세한 가이드입니다.
Packages 디렉토리는 여러 애플리케이션에서 공유되는 재사용 가능한 라이브러리들을 포함합니다.

## Packages 디렉토리 개요

`packages/` 디렉토리는 다음과 같은 특징을 가집니다:

- 재사용 가능한 라이브러리 및 유틸리티
- 여러 앱에서 공유되는 코드
- 독립적으로 테스트 및 빌드 가능
- NPM 패키지로 퍼블리시 가능 (선택사항)

## 일반적인 Packages 구조

```
packages/
├── ui-components/           # UI 컴포넌트 라이브러리
│   ├── src/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── styles/
│   │   └── index.ts
│   ├── package.json
│   ├── vite.lib.config.ts
│   └── tsconfig.json
├── shared-lib/              # 공유 유틸리티 및 비즈니스 로직
│   ├── src/
│   │   ├── api/
│   │   ├── utils/
│   │   ├── constants/
│   │   └── index.ts
│   ├── package.json
│   └── vite.lib.config.ts
├── types/                   # 공유 타입 정의
│   ├── src/
│   │   ├── api.ts
│   │   ├── common.ts
│   │   └── index.ts
│   ├── package.json
│   └── tsconfig.json
├── config/                  # 공유 설정 (별도 config/ 디렉토리로 이동 권장)
│   ├── eslint-config/
│   ├── typescript-config/
│   └── vite-config/
└── tools/                   # 개발 도구
    ├── build-scripts/
    └── dev-tools/
```

## 패키지별 설정

### UI 컴포넌트 라이브러리

```json
{
  "name": "@project/ui-components",
  "version": "1.0.0",
  "description": "Shared UI Components Library",
  "private": false,
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./styles": {
      "import": "./dist/styles.css"
    }
  },
  "files": ["dist", "README.md"],
  "sideEffects": ["**/*.css"],
  "scripts": {
    "build": "vite build --config vite.lib.config.ts",
    "build:watch": "vite build --config vite.lib.config.ts --watch",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "lint": "eslint src --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint src --ext ts,tsx --fix",
    "format": "prettier --check \"src/**/*.{ts,tsx,json,md}\"",
    "format:fix": "prettier --write \"src/**/*.{ts,tsx,json,md}\"",
    "typecheck": "tsc --noEmit",
    "storybook": "storybook dev -p 6006",
    "build-storybook": "storybook build",
    "clean": "rimraf dist node_modules .turbo",
    "prebuild": "rimraf dist",
    "prepublishOnly": "pnpm test && pnpm build"
  },
  "peerDependencies": {
    "react": ">=18",
    "react-dom": ">=18"
  },
  "devDependencies": {
    "@storybook/addon-essentials": "^7.6.0",
    "@storybook/addon-interactions": "^7.6.0",
    "@storybook/addon-links": "^7.6.0",
    "@storybook/blocks": "^7.6.0",
    "@storybook/react": "^7.6.0",
    "@storybook/react-vite": "^7.6.0",
    "@storybook/testing-library": "^0.2.2",
    "@testing-library/jest-dom": "^6.0.0",
    "@testing-library/react": "^14.0.0",
    "@testing-library/user-event": "^14.0.0",
    "@types/react": "^19.1.6",
    "@types/react-dom": "^19.1.5",
    "@vitejs/plugin-react": "^4.3.1",
    "@vitest/coverage-v8": "^1.0.0",
    "@vitest/ui": "^1.0.0",
    "jsdom": "^23.0.0",
    "react": "^19.1.0",
    "react-dom": "^19.1.0",
    "storybook": "^7.6.0",
    "typescript": "~5.8.3",
    "vite": "^6.3.5",
    "vite-plugin-dts": "^3.9.1",
    "vitest": "^1.0.0"
  }
}
```

### 공유 라이브러리

```json
{
  "name": "@project/shared-lib",
  "version": "1.0.0",
  "description": "Shared Library with utilities and business logic",
  "private": false,
  "main": "./dist/index.cjs",
  "module": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./api": {
      "types": "./dist/api.d.ts",
      "import": "./dist/api.js",
      "require": "./dist/api.cjs"
    },
    "./utils": {
      "types": "./dist/utils.d.ts",
      "import": "./dist/utils.js",
      "require": "./dist/utils.cjs"
    }
  },
  "files": ["dist", "README.md"],
  "sideEffects": false,
  "scripts": {
    "build": "vite build --config vite.lib.config.ts",
    "build:watch": "vite build --config vite.lib.config.ts --watch",
    "test": "vitest",
    "test:coverage": "vitest run --coverage",
    "lint": "eslint src --ext ts --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint src --ext ts --fix",
    "format": "prettier --check \"src/**/*.{ts,json,md}\"",
    "format:fix": "prettier --write \"src/**/*.{ts,json,md}\"",
    "typecheck": "tsc --noEmit",
    "clean": "rimraf dist node_modules .turbo",
    "prebuild": "rimraf dist",
    "prepublishOnly": "pnpm test && pnpm build"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "@vitest/coverage-v8": "^1.0.0",
    "typescript": "~5.8.3",
    "vite": "^6.3.5",
    "vite-plugin-dts": "^3.9.1",
    "vitest": "^1.0.0"
  }
}
```

### 타입 정의 패키지

```json
{
  "name": "@project/types",
  "version": "1.0.0",
  "description": "Shared TypeScript type definitions",
  "private": false,
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./api": {
      "types": "./dist/api.d.ts"
    },
    "./common": {
      "types": "./dist/common.d.ts"
    }
  },
  "files": ["dist", "README.md"],
  "sideEffects": false,
  "scripts": {
    "build": "tsc -p tsconfig.build.json",
    "build:watch": "tsc -p tsconfig.build.json --watch",
    "lint": "eslint src --ext ts --report-unused-disable-directives --max-warnings 0",
    "lint:fix": "eslint src --ext ts --fix",
    "typecheck": "tsc --noEmit",
    "clean": "rimraf dist node_modules .turbo",
    "prebuild": "rimraf dist"
  },
  "devDependencies": {
    "typescript": "~5.8.3"
  }
}
```

## Vite 라이브러리 빌드 설정

```typescript
// packages/ui-components/vite.lib.config.ts
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
        index: resolve(__dirname, "src/index.ts"),
        components: resolve(__dirname, "src/components/index.ts"),
        hooks: resolve(__dirname, "src/hooks/index.ts"),
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

## 패키지별 구조 예시

### UI 컴포넌트 라이브러리 구조

```
packages/ui-components/
├── src/
│   ├── components/
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.test.tsx
│   │   │   ├── Button.stories.tsx
│   │   │   ├── Button.module.css
│   │   │   └── index.ts
│   │   ├── Card/
│   │   ├── Modal/
│   │   └── index.ts
│   ├── hooks/
│   │   ├── useLocalStorage.ts
│   │   ├── useDebounce.ts
│   │   └── index.ts
│   ├── styles/
│   │   ├── globals.css
│   │   ├── variables.css
│   │   └── index.ts
│   └── index.ts
├── .storybook/
├── package.json
├── vite.lib.config.ts
└── tsconfig.json
```

### 공유 라이브러리 구조

```
packages/shared-lib/
├── src/
│   ├── api/
│   │   ├── client.ts
│   │   ├── endpoints.ts
│   │   ├── interceptors.ts
│   │   └── index.ts
│   ├── utils/
│   │   ├── date.ts
│   │   ├── format.ts
│   │   ├── validation.ts
│   │   └── index.ts
│   ├── constants/
│   │   ├── api.ts
│   │   ├── app.ts
│   │   └── index.ts
│   ├── store/
│   │   ├── createStore.ts
│   │   └── index.ts
│   └── index.ts
├── package.json
├── vite.lib.config.ts
└── tsconfig.json
```

## 컴포넌트 작성 가이드

### 컴포넌트 예시

```typescript
// packages/ui-components/src/components/Button/Button.tsx
import React from "react";
import styles from "./Button.module.css";

export interface ButtonProps {
  variant?: "primary" | "secondary" | "danger";
  size?: "small" | "medium" | "large";
  disabled?: boolean;
  loading?: boolean;
  children: React.ReactNode;
  onClick?: () => void;
}

export const Button: React.FC<ButtonProps> = ({
  variant = "primary",
  size = "medium",
  disabled = false,
  loading = false,
  children,
  onClick,
}) => {
  const className = [
    styles.button,
    styles[variant],
    styles[size],
    disabled && styles.disabled,
    loading && styles.loading,
  ]
    .filter(Boolean)
    .join(" ");

  return (
    <button
      className={className}
      disabled={disabled || loading}
      onClick={onClick}
    >
      {loading && <span className={styles.spinner} />}
      {children}
    </button>
  );
};
```

### 스토리북 스토리

```typescript
// packages/ui-components/src/components/Button/Button.stories.tsx
import type { Meta, StoryObj } from "@storybook/react";
import { Button } from "./Button";

const meta: Meta<typeof Button> = {
  title: "Components/Button",
  component: Button,
  parameters: {
    layout: "centered",
  },
  tags: ["autodocs"],
  argTypes: {
    variant: {
      control: "select",
      options: ["primary", "secondary", "danger"],
    },
    size: {
      control: "select",
      options: ["small", "medium", "large"],
    },
  },
};

export default meta;
type Story = StoryObj<typeof meta>;

export const Primary: Story = {
  args: {
    variant: "primary",
    children: "Button",
  },
};

export const Secondary: Story = {
  args: {
    variant: "secondary",
    children: "Button",
  },
};

export const Loading: Story = {
  args: {
    variant: "primary",
    loading: true,
    children: "Loading...",
  },
};
```

### 테스트 예시

```typescript
// packages/ui-components/src/components/Button/Button.test.tsx
import { render, screen, fireEvent } from "@testing-library/react";
import { describe, it, expect, vi } from "vitest";
import { Button } from "./Button";

describe("Button", () => {
  it("renders with children", () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole("button")).toHaveTextContent("Click me");
  });

  it("calls onClick when clicked", () => {
    const handleClick = vi.fn();
    render(<Button onClick={handleClick}>Click me</Button>);

    fireEvent.click(screen.getByRole("button"));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it("is disabled when disabled prop is true", () => {
    render(<Button disabled>Click me</Button>);
    expect(screen.getByRole("button")).toBeDisabled();
  });

  it("shows loading state", () => {
    render(<Button loading>Loading...</Button>);
    const button = screen.getByRole("button");
    expect(button).toBeDisabled();
    expect(button).toHaveClass("loading");
  });
});
```

## 유틸리티 함수 예시

```typescript
// packages/shared-lib/src/utils/date.ts
export const formatDate = (
  date: Date,
  format: "short" | "long" = "short"
): string => {
  const options: Intl.DateTimeFormatOptions =
    format === "long"
      ? { year: "numeric", month: "long", day: "numeric" }
      : { year: "numeric", month: "2-digit", day: "2-digit" };

  return new Intl.DateTimeFormat("ko-KR", options).format(date);
};

export const isToday = (date: Date): boolean => {
  const today = new Date();
  return date.toDateString() === today.toDateString();
};

export const addDays = (date: Date, days: number): Date => {
  const result = new Date(date);
  result.setDate(result.getDate() + days);
  return result;
};
```

## API 클라이언트 예시

```typescript
// packages/shared-lib/src/api/client.ts
import type { ApiResponse } from "@project/types";

class ApiClient {
  private baseURL: string;
  private defaultHeaders: Record<string, string>;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      "Content-Type": "application/json",
    };
  }

  async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<ApiResponse<T>> {
    const url = `${this.baseURL}${endpoint}`;
    const config: RequestInit = {
      ...options,
      headers: {
        ...this.defaultHeaders,
        ...options.headers,
      },
    };

    try {
      const response = await fetch(url, config);
      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.message || "API request failed");
      }

      return data;
    } catch (error) {
      console.error("API request error:", error);
      throw error;
    }
  }

  get<T>(endpoint: string): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, { method: "GET" });
  }

  post<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, {
      method: "POST",
      body: JSON.stringify(data),
    });
  }

  put<T>(endpoint: string, data: any): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, {
      method: "PUT",
      body: JSON.stringify(data),
    });
  }

  delete<T>(endpoint: string): Promise<ApiResponse<T>> {
    return this.request<T>(endpoint, { method: "DELETE" });
  }
}

export const apiClient = new ApiClient(process.env.VITE_API_URL || "/api");
```

## 테스트 설정

### Vitest 설정

```typescript
// packages/ui-components/vitest.config.ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: "jsdom",
    setupFiles: ["./src/test/setup.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      exclude: [
        "node_modules/",
        "src/test/",
        "**/*.stories.tsx",
        "**/*.test.tsx",
      ],
    },
  },
});
```

### 테스트 설정 파일

```typescript
// packages/ui-components/src/test/setup.ts
import "@testing-library/jest-dom";
import { expect, afterEach } from "vitest";
import { cleanup } from "@testing-library/react";

// runs a cleanup after each test case
afterEach(() => {
  cleanup();
});
```

## 퍼블리시 및 배포

### NPM 퍼블리시

```bash
# 패키지 빌드
pnpm build --filter=@project/ui-components

# 패키지 퍼블리시
pnpm publish --filter=@project/ui-components
```

### Changeset을 사용한 버전 관리

```bash
# 변경사항 추가
pnpm changeset

# 버전 업데이트
pnpm changeset version

# 퍼블리시
pnpm changeset publish
```

## 설정 관리

**⚠️ 중요**: ESLint, TypeScript, Prettier 등의 설정 파일은 **별도 `config/` 디렉토리**에서 관리하는 것을 권장합니다.

자세한 내용은 [설정 파일 관리 가이드](./config.md)를 참고하세요.

## 개발 가이드라인

### 패키지 설계 원칙

1. **단일 책임**: 각 패키지는 하나의 명확한 목적을 가져야 함
2. **최소 의존성**: 외부 의존성을 최소화
3. **트리 쉐이킹**: 사용하지 않는 코드는 번들에 포함되지 않도록 설계
4. **타입 안정성**: 모든 공개 API는 타입을 제공

### 코딩 컨벤션

1. **Export 규칙**: named export 사용 (default export 금지)
2. **파일 네이밍**: PascalCase for components, camelCase for utilities
3. **디렉토리 구조**: 기능별로 명확하게 분리
4. **문서화**: 모든 공개 API는 JSDoc으로 문서화

### 호환성 관리

1. **Peer Dependencies**: React 등 호스트 앱에서 제공되는 라이브러리
2. **버전 호환성**: Semantic Versioning 준수
3. **Breaking Changes**: 메이저 버전에서만 허용

## 문제 해결

### 빌드 이슈

```bash
# 의존성 이슈
pnpm install --filter=@project/package-name

# 타입 체크
pnpm typecheck --filter=@project/package-name

# 캐시 클리어
pnpm clean --filter=@project/package-name
```

### 순환 의존성 방지

- 패키지 간 의존성 그래프를 명확히 관리
- `packages/types`를 최상위 레벨로 유지
- 비즈니스 로직과 UI 컴포넌트 분리
