# Vitest 테스트 전략 가이드

라이브러리 개발을 위한 체계적인 테스트 전략과 워크플로우를 다룹니다.

## 개요

이 문서는 [기본 Vitest 설정](./vitest.md)을 기반으로 한 실용적인 테스트 전략을 제시합니다. 단위 테스트와 통합 테스트의 균형있는 접근법을 통해 효율적인 개발 워크플로우를 구축할 수 있습니다.

## 테스트 전략 개요

### 이중 테스트 접근법

라이브러리 개발에서는 두 가지 레벨의 테스트가 필요합니다:

1. **단위 테스트 (Vitest)**: 라이브러리 내부 로직 검증
2. **통합 테스트 (데모 앱)**: 실제 사용 환경에서의 동작 검증

### 왜 이런 구조인가?

| 테스트 유형     | 목적                           | 장점                                     | 단점                       |
| --------------- | ------------------------------ | ---------------------------------------- | -------------------------- |
| **단위 테스트** | 개별 함수/컴포넌트 로직 검증   | 빠른 피드백, 격리된 테스트, CI/CD 친화적 | 실제 사용 환경과 차이      |
| **통합 테스트** | 번들링, 타입 정의, 실사용 검증 | 실제 환경과 동일, 예제 제공              | 설정 복잡, 상대적으로 느림 |

## 프로젝트 구조

### 권장 디렉토리 구성

```
project-root/
├── packages/
│   └── shared-lib/            # 라이브러리 패키지
│       ├── src/
│       │   ├── components/
│       │   ├── hooks/
│       │   └── utils/
│       ├── tests/             # 단위 테스트만
│       │   ├── setup.ts
│       │   └── __tests__/
│       ├── vitest.config.ts
│       └── package.json
├── apps/
│   ├── main-app/              # 메인 애플리케이션
│   └── shared-lib-demo/       # 라이브러리 데모/통합 테스트 앱
│       ├── src/
│       │   ├── examples/      # 사용 예제들
│       │   └── tests/         # 통합 테스트
│       ├── vite.config.ts
│       └── package.json
└── turbo.json
```

### 패키지별 역할 분담

#### 라이브러리 패키지 (`packages/shared-lib`)

```json
{
  "name": "@project/shared-lib",
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "build": "vite build",
    "build:watch": "vite build --watch"
  },
  "devDependencies": {
    "vitest": "^1.0.0",
    "@testing-library/react": "^14.0.0",
    "@testing-library/jest-dom": "^6.0.0"
  }
}
```

#### 데모 앱 (`apps/shared-lib-demo`)

```json
{
  "name": "@project/shared-lib-demo",
  "private": true,
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "test:integration": "vitest",
    "preview": "vite preview"
  },
  "dependencies": {
    "@project/shared-lib": "workspace:*",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  }
}
```

## 단위 테스트 설정

### 라이브러리용 vitest.config.ts

```typescript
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import { resolve } from "path";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./tests/setup.ts"],
    globals: true,
    coverage: {
      provider: "v8",
      reporter: ["text", "json", "html"],
      exclude: [
        "node_modules/",
        "tests/",
        "**/*.d.ts",
        "**/*.config.*",
        "**/index.ts", // re-export 파일
      ],
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
  resolve: {
    alias: {
      "@": resolve(__dirname, "./src"),
    },
  },
});
```

### 테스트 설정 파일

```typescript
// tests/setup.ts
import "@testing-library/jest-dom";
import { cleanup } from "@testing-library/react";
import { afterEach } from "vitest";

// 각 테스트 후 자동 정리
afterEach(() => {
  cleanup();
});

// 브라우저 API 모킹
global.ResizeObserver = class ResizeObserver {
  observe() {}
  unobserve() {}
  disconnect() {}
};

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

## 테스트 작성 패턴

### 컴포넌트 테스트 예시

```typescript
// tests/__tests__/Button.test.tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, it, expect, vi } from "vitest";
import { Button } from "../src/components/Button";

describe("Button", () => {
  it("renders with correct text", () => {
    render(<Button>Click me</Button>);
    expect(
      screen.getByRole("button", { name: "Click me" })
    ).toBeInTheDocument();
  });

  it("handles click events", async () => {
    const user = userEvent.setup();
    const handleClick = vi.fn();

    render(<Button onClick={handleClick}>Click me</Button>);

    await user.click(screen.getByRole("button"));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it("applies variant styles correctly", () => {
    render(<Button variant="primary">Primary</Button>);
    expect(screen.getByRole("button")).toHaveClass("button--primary");
  });
});
```

### 훅 테스트 예시

```typescript
// tests/__tests__/useCounter.test.ts
import { renderHook, act } from "@testing-library/react";
import { describe, it, expect } from "vitest";
import { useCounter } from "../src/hooks/useCounter";

describe("useCounter", () => {
  it("should initialize with default value", () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it("should increment counter", () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it("should accept initial value", () => {
    const { result } = renderHook(() => useCounter(5));
    expect(result.current.count).toBe(5);
  });
});
```

## 통합 테스트 설정

### 데모 앱 구조

```typescript
// apps/shared-lib-demo/src/App.tsx
import React from "react";
import { Button, useCounter } from "@project/shared-lib";

function App() {
  const { count, increment, decrement } = useCounter(0);

  return (
    <div className="app">
      <h1>Shared Library Demo</h1>

      <section>
        <h2>Button Component</h2>
        <Button variant="primary" onClick={increment}>
          Increment ({count})
        </Button>
        <Button variant="secondary" onClick={decrement}>
          Decrement
        </Button>
      </section>

      <section>
        <h2>Hook Examples</h2>
        <p>Current count: {count}</p>
      </section>
    </div>
  );
}

export default App;
```

### 통합 테스트 예시

```typescript
// apps/shared-lib-demo/src/tests/integration.test.tsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { describe, it, expect } from "vitest";
import App from "../App";

describe("Library Integration Tests", () => {
  it("should integrate Button and useCounter correctly", async () => {
    const user = userEvent.setup();
    render(<App />);

    // 초기 상태 확인
    expect(screen.getByText("Current count: 0")).toBeInTheDocument();
    expect(screen.getByText("Increment (0)")).toBeInTheDocument();

    // 버튼 클릭 후 상태 변경 확인
    await user.click(screen.getByText("Increment (0)"));

    expect(screen.getByText("Current count: 1")).toBeInTheDocument();
    expect(screen.getByText("Increment (1)")).toBeInTheDocument();
  });

  it("should handle multiple interactions", async () => {
    const user = userEvent.setup();
    render(<App />);

    const incrementBtn = screen.getByText(/Increment/);
    const decrementBtn = screen.getByText("Decrement");

    // 여러 번 클릭
    await user.click(incrementBtn);
    await user.click(incrementBtn);
    await user.click(decrementBtn);

    expect(screen.getByText("Current count: 1")).toBeInTheDocument();
  });
});
```

## 개발 워크플로우

### 일반적인 개발 프로세스

```bash
# 1. 라이브러리 개발 및 테스트
cd packages/shared-lib

# 단위 테스트 watch 모드
pnpm test

# 새 기능 개발 후 테스트 실행
pnpm test:coverage

# 2. 라이브러리 빌드 (데모 앱에서 사용하기 위해)
pnpm build

# 3. 데모 앱에서 통합 테스트
cd ../../apps/shared-lib-demo
pnpm dev
```

### 실시간 개발 워크플로우

두 개의 터미널을 사용한 효율적인 개발:

```bash
# 터미널 1: 라이브러리 watch 빌드
cd packages/shared-lib
pnpm build:watch

# 터미널 2: 데모 앱 실행
cd apps/shared-lib-demo
pnpm dev
```

이렇게 하면 라이브러리 코드 변경 시 자동으로 빌드되고, 데모 앱에서 즉시 확인할 수 있습니다.

### CI/CD 파이프라인

테스트를 CI/CD 파이프라인에 통합하는 상세한 방법은 별도 가이드를 참고하세요:

 **[../cicd/quality-pipeline.md](../cicd/quality-pipeline.md)** - 품질 관리 파이프라인 가이드

### 테스트 자동화 전략

- **[GitHub Actions](../cicd/github-actions.md)**: GitHub 워크플로우에서 테스트 실행
- **[Azure Pipelines](../cicd/azure-pipelines.md)**: Azure DevOps에서 테스트 관리

## 품질 관리

### 테스트 커버리지 목표

| 유형          | 최소 커버리지 | 권장 커버리지 |
| ------------- | ------------- | ------------- |
| 유틸리티 함수 | 90%           | 95%           |
| 커스텀 훅     | 85%           | 90%           |
| 컴포넌트      | 80%           | 85%           |
| 전체 프로젝트 | 80%           | 85%           |

### pre-commit 훅 설정

```json
// .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# 변경된 라이브러리가 있는 경우만 테스트 실행
if git diff --cached --name-only | grep -q "packages/shared-lib/"; then
  echo "Running library tests..."
  pnpm --filter @project/shared-lib test:run
fi
```

## 모범 사례

### 1. 테스트 격리 원칙

```typescript
//  좋지 않은 예 - 전역 상태 공유
let globalCounter = 0;

describe("Counter tests", () => {
  it("increments", () => {
    globalCounter++; // 다른 테스트에 영향
    expect(globalCounter).toBe(1);
  });
});

//  좋은 예 - 각 테스트 독립
describe("Counter tests", () => {
  it("increments", () => {
    const { result } = renderHook(() => useCounter(0));
    act(() => result.current.increment());
    expect(result.current.count).toBe(1);
  });
});
```

### 2. 의미 있는 테스트 작성

```typescript
//  구현 세부사항 테스트
it("calls useState with initial value", () => {
  const setStateSpy = vi.spyOn(React, "useState");
  renderHook(() => useCounter(5));
  expect(setStateSpy).toHaveBeenCalledWith(5);
});

//  동작 결과 테스트
it("initializes with provided value", () => {
  const { result } = renderHook(() => useCounter(5));
  expect(result.current.count).toBe(5);
});
```

### 3. 적절한 테스트 범위

```typescript
//  핵심 기능 집중 테스트
describe("useLocalStorage", () => {
  it("persists value to localStorage", () => {
    const { result } = renderHook(() => useLocalStorage("key", "initial"));

    act(() => {
      result.current[1]("updated");
    });

    expect(localStorage.getItem("key")).toBe('"updated"');
  });

  it("handles invalid JSON gracefully", () => {
    localStorage.setItem("key", "invalid json");
    const { result } = renderHook(() => useLocalStorage("key", "fallback"));

    expect(result.current[0]).toBe("fallback");
  });
});
```

## 문제 해결

### 자주 발생하는 이슈들

1. **데모 앱에서 라이브러리 변경사항이 반영되지 않음**

   ```bash
   # 라이브러리 재빌드
   cd packages/shared-lib
   pnpm build

   # 또는 watch 모드 사용
   pnpm build:watch
   ```

2. **타입 정의 파일 문제**

   ```typescript
   // packages/shared-lib/tsconfig.json
   {
     "compilerOptions": {
       "declaration": true,
       "declarationMap": true,
       "outDir": "./dist"
     }
   }
   ```

3. **의존성 버전 충돌**

   ```json
   // 루트 package.json에서 통일
   {
     "peerDependencies": {
       "react": "^18.0.0",
       "react-dom": "^18.0.0"
     }
   }
   ```

## 성능 최적화

### 테스트 실행 속도 향상

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    // 병렬 실행 최적화
    pool: "threads",
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

### 빌드 최적화

```typescript
// vite.config.ts (라이브러리용)
export default defineConfig({
  build: {
    watch: process.env.NODE_ENV === "development" ? {} : null,
    rollupOptions: {
      external: ["react", "react-dom"],
    },
  },
});
```

## 결론

이 테스트 전략을 통해 얻는 이익:

- **빠른 피드백**: 단위 테스트로 즉시 문제 발견
- **실제 환경 검증**: 데모 앱으로 실사용 확인
- **문서화 효과**: 데모 앱이 사용 예제 역할
- **유지보수성**: 체계적인 테스트로 안전한 리팩토링

## 관련 문서

- [vitest.md](./vitest.md) - 기본 Vitest 설정
- [vitest-options.md](./vitest-options.md) - 고급 옵션 및 대안
- [../quality/README.md](../quality/README.md) - 코드 품질 관리
- [../monorepo/README.md](../monorepo/README.md) - 모노레포 구성
