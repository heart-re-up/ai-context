# Vitest 테스트 설정 가이드

Vitest를 사용한 라이브러리 테스트 환경 구성과 설정 방법에 대한 가이드입니다.

## 개요

Vitest는 Vite 기반의 빠른 테스트 프레임워크로, 라이브러리 개발 시 다음과 같은 장점을 제공합니다:

- **빠른 실행**: Vite의 빠른 빌드 시스템 활용
- **ES 모듈 지원**: 최신 JavaScript 기능 네이티브 지원
- **Hot Reload**: 테스트 파일 변경 시 즉시 재실행
- **TypeScript 지원**: 별도 설정 없이 TypeScript 테스트 작성 가능
- **Jest 호환성**: Jest API와 유사한 인터페이스

## 테스트 환경 이해

### 헤드리스 DOM 환경이란?

브라우저 없이 DOM API를 시뮬레이션하는 환경입니다.

**왜 필요한가?**

- 실제 브라우저는 무겁고 느려서 테스트에 부적합
- UI 없이 DOM 조작과 렌더링 로직만 테스트하면 충분
- CI/CD 환경에서는 GUI가 없어 실제 브라우저 실행 불가

### 권장 환경: jsdom

React 컴포넌트 테스트를 위해서는 **jsdom**을 권장합니다:

-  **완전한 DOM API** 지원
-  **Jest 호환성** - 기존 코드 마이그레이션 용이
-  **안정성** - 충분히 검증된 환경
-  **풍부한 커뮤니티** 지원

> **다른 환경 옵션들** (happy-dom, 브라우저 테스트 등)이 궁금하다면 [고급 옵션 가이드](./vitest-other-options.md)를 참고하세요.

## 기본 설정

### 설치

```bash
# 필수 라이브러리 설치
pnpm add -D vitest @vitest/ui jsdom
pnpm add -D @testing-library/react @testing-library/jest-dom @testing-library/user-event
```

### 라이브러리 역할

| 라이브러리                    | 역할                       | 필수여부             |
| ----------------------------- | -------------------------- | -------------------- |
| `vitest`                      | 테스트 실행 엔진           | 필수                 |
| `jsdom`                       | DOM 환경 제공              | React 테스트 시 필수 |
| `@testing-library/react`      | React 컴포넌트 렌더링/쿼리 | React 테스트 시 필수 |
| `@testing-library/jest-dom`   | DOM 관련 assertion 확장    | 권장                 |
| `@testing-library/user-event` | 사용자 상호작용 시뮬레이션 | 권장                 |

> **더 많은 옵션과 대안들**이 궁금하다면 [고급 옵션 가이드](./vitest-other-options.md)를 참고하세요.

### 기본 vitest.config.ts

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import { resolve } from "path";

export default defineConfig({
  plugins: [react()], // React JSX 변환
  test: {
    environment: "jsdom", // DOM API 사용
    setupFiles: ["./src/test/setup.ts"], // 전역 설정
    globals: true, // describe, it, expect 전역 사용
  },
  resolve: {
    alias: {
      "@": resolve(__dirname, "src"), // 절대 경로 import
    },
  },
});
```

### 테스트 환경 설정

```typescript
// src/test/setup.ts
import "@testing-library/jest-dom"; // DOM matcher 확장

// 브라우저 API 폴리필
global.ResizeObserver = class ResizeObserver {
  observe() {}
  unobserve() {}
  disconnect() {}
};

// CSS 미디어 쿼리 모킹
Object.defineProperty(window, "matchMedia", {
  writable: true,
  value: (query: string) => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: () => {},
    removeListener: () => {},
    addEventListener: () => {},
    removeEventListener: () => {},
    dispatchEvent: () => {},
  }),
});
```

## 라이브러리별 테스트 설정

### React 컴포넌트 라이브러리

```typescript
// vitest.config.ts
export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./src/test/setup.ts"],
    globals: true,
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      exclude: [
        "node_modules/",
        "src/test/",
        "**/*.d.ts",
        "**/*.config.*",
        "**/index.ts", // re-export 파일
      ],
    },
  },
});
```

### 유틸리티 라이브러리

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    environment: "node", // DOM 불필요
    globals: true,
    coverage: {
      provider: "v8",
      reporter: ["text", "json"],
      thresholds: {
        global: {
          branches: 80,
          functions: 80,
          lines: 80,
          statements: 80,
        },
      },
    },
  },
});
```

### 훅 라이브러리

```typescript
// vitest.config.ts
export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./src/test/setup.ts"],
    globals: true,
  },
});

// src/test/setup.ts
import "@testing-library/jest-dom";
import { cleanup } from "@testing-library/react";
import { afterEach } from "vitest";

afterEach(() => {
  cleanup();
});
```

## 테스트 작성 패턴

### 컴포넌트 테스트

```typescript
// src/components/Button/Button.test.tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, it, expect, vi } from "vitest";
import { Button } from "./Button";

describe("Button", () => {
  it("renders with correct text", () => {
    render(<Button>Click me</Button>);
    expect(
      screen.getByRole("button", { name: "Click me" })
    ).toBeInTheDocument();
  });

  it("calls onClick when clicked", async () => {
    const user = userEvent.setup();
    const handleClick = vi.fn();

    render(<Button onClick={handleClick}>Click me</Button>);

    await user.click(screen.getByRole("button"));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it("supports different variants", () => {
    render(<Button variant="primary">Primary</Button>);
    expect(screen.getByRole("button")).toHaveClass("button--primary");
  });
});
```

### 훅 테스트

```typescript
// src/hooks/useLocalStorage/useLocalStorage.test.ts
import { renderHook, act } from "@testing-library/react";
import { describe, it, expect, beforeEach, vi } from "vitest";
import { useLocalStorage } from "./useLocalStorage";

describe("useLocalStorage", () => {
  beforeEach(() => {
    localStorage.clear();
    vi.clearAllMocks();
  });

  it("returns initial value when no stored value", () => {
    const { result } = renderHook(() => useLocalStorage("key", "initial"));
    expect(result.current[0]).toBe("initial");
  });

  it("updates localStorage when value changes", () => {
    const { result } = renderHook(() => useLocalStorage("key", "initial"));

    act(() => {
      result.current[1]("updated");
    });

    expect(localStorage.getItem("key")).toBe('"updated"');
    expect(result.current[0]).toBe("updated");
  });
});
```

### 유틸리티 함수 테스트

```typescript
// src/utils/formatDate/formatDate.test.ts
import { describe, it, expect } from "vitest";
import { formatDate } from "./formatDate";

describe("formatDate", () => {
  it("formats date correctly", () => {
    const date = new Date("2023-12-25T00:00:00Z");
    expect(formatDate(date, "YYYY-MM-DD")).toBe("2023-12-25");
  });

  it("handles invalid dates", () => {
    const invalidDate = new Date("invalid");
    expect(formatDate(invalidDate, "YYYY-MM-DD")).toBe("Invalid Date");
  });

  it("supports custom formats", () => {
    const date = new Date("2023-12-25T15:30:00Z");
    expect(formatDate(date, "MM/DD/YYYY HH:mm")).toBe("12/25/2023 15:30");
  });
});
```

## 커버리지 설정

### 상세 커버리지 설정

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    coverage: {
      provider: "v8",
      reporter: ["text", "json-summary", "html", "lcov"],
      reportsDirectory: "./coverage",
      exclude: [
        "coverage/**",
        "dist/**",
        "**/*.d.ts",
        "**/*.config.*",
        "**/*.test.*",
        "**/*.spec.*",
        "**/test/**",
        "**/tests/**",
        "**/__tests__/**",
        "**/node_modules/**",
        "**/index.ts", // re-export만 하는 파일
        "**/*.stories.*", // Storybook 파일
      ],
      include: ["src/**/*.{ts,tsx}"],
      all: true,
      thresholds: {
        global: {
          branches: 80,
          functions: 80,
          lines: 80,
          statements: 80,
        },
      },
    },
  },
});
```

## 패키지 스크립트 설정

### package.json

```json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:watch": "vitest watch",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest run --coverage",
    "test:coverage:ui": "vitest --ui --coverage"
  }
}
```

### 워크플로우별 실행

```bash
# 개발 중 - watch 모드
pnpm test

# CI/CD - 한 번만 실행
pnpm test:run

# 커버리지 확인
pnpm test:coverage

# UI로 테스트 결과 확인
pnpm test:ui
```

## CI/CD 통합

### GitHub Actions

```yaml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"

      - run: pnpm install --frozen-lockfile

      - name: Run tests
        run: pnpm test:coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

## 성능 최적화

### 테스트 실행 속도 향상

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    // 빠른 실행을 위한 설정
    passWithNoTests: true,
    watch: false, // CI에서는 watch 모드 비활성화

    // 메모리 사용량 최적화
    poolOptions: {
      threads: {
        maxThreads: 4,
        minThreads: 1,
      },
    },

    // 불필요한 파일 제외
    exclude: ["**/node_modules/**", "**/dist/**", "**/*.e2e.test.*"],
  },
});
```

## 문제 해결

### 자주 발생하는 이슈

1. **JSDOM 환경 문제**

   ```typescript
   // vitest.config.ts
   test: {
     environment: "jsdom",
     setupFiles: ["./src/test/setup.ts"],
   }
   ```

2. **모듈 해석 문제**

   ```typescript
   resolve: {
     alias: {
       "@": resolve(__dirname, "src"),
     },
   }
   ```

3. **TypeScript 타입 문제**

   ```typescript
   // tsconfig.json
   {
     "compilerOptions": {
       "types": ["vitest/globals", "@testing-library/jest-dom"]
     }
   }
   ```

4. **React 18 호환성**

   ```typescript
   // src/test/setup.ts
   import { configure } from "@testing-library/react";

   configure({
     asyncUtilTimeout: 2000,
   });
   ```

## 모범 사례

1. **테스트 격리**: 각 테스트는 독립적으로 실행되어야 함
2. **명확한 테스트명**: 테스트가 무엇을 검증하는지 명확히 작성
3. **AAA 패턴**: Arrange, Act, Assert 구조로 테스트 작성
4. **적절한 커버리지**: 100%보다는 의미 있는 테스트에 집중
5. **빠른 피드백**: watch 모드를 활용한 개발 워크플로우

## 참고 자료

- [Vitest 공식 문서](https://vitest.dev/)
- [Testing Library 공식 문서](https://testing-library.com/)
- [Jest DOM Matchers](https://github.com/testing-library/jest-dom)
- [User Event 가이드](https://testing-library.com/docs/user-event/intro/)

## 관련 문서

- [vitest-strategy.md](./vitest-strategy.md) - **테스트 전략 및 워크플로우**
- [vitest-options.md](./vitest-options.md) - 고급 옵션 및 대안 환경
- [../quality/README.md](../quality/README.md) - 코드 품질 도구
- [../vite/vite-lib.md](../vite/vite-lib.md) - 라이브러리 빌드 설정
- [../vite/vite-dev.md](../vite/vite-dev.md) - 개발 환경 설정
