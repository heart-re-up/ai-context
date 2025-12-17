# Vitest 고급 옵션 및 대안 환경

이 문서는 기본적인 jsdom + jest-dom 조합 외의 다양한 테스트 환경과 도구 옵션들을 다룹니다.

## 개요

[vitest.md](./vitest.md)에서는 가장 실용적이고 안정적인 조합을 다루었습니다. 이 문서는 특수한 요구사항이나 성능 최적화가 필요한 경우의 대안들을 설명합니다.

## 테스트 환경 대안들

### Happy-DOM vs jsdom 비교

| 환경          | 장점                                  | 단점                                | 적합한 경우                                             |
| ------------- | ------------------------------------- | ----------------------------------- | ------------------------------------------------------- |
| **jsdom**     | Jest 호환, 안정성, **완전한 DOM API** | 상대적으로 느림, 메모리 사용량 높음 | React 컴포넌트, 기존 Jest 프로젝트, **복잡한 DOM 조작** |
| **happy-dom** | 매우 빠름, 가벼움                     | **중요한 기능 제한**, CSS 계산 미흡 | 간단한 컴포넌트, 성능 우선 프로젝트                     |

### Happy-DOM의 주요 제약사항

**CSS 관련 제한**

- CSS 스타일 계산이 부정확하거나 누락
- `getComputedStyle()` 결과가 실제 브라우저와 다름
- CSS 애니메이션, 트랜지션 미지원
- Flexbox, Grid 레이아웃 계산 오류

**DOM API 제한**

- 일부 최신 DOM API 미구현
- Shadow DOM 지원 제한적
- 이벤트 시스템이 단순화됨
- 드래그 앤 드롭 API 없음

**브라우저 API 제한**

- Intersection Observer, Resize Observer 등 누락
- Canvas, WebGL 관련 API 없음
- File API, Clipboard API 제한적
- Web Components 지원 불완전

### 환경별 설정

```typescript
// Happy-DOM 환경 - 성능 우선
export default defineConfig({
  test: {
    environment: "happy-dom",
    setupFiles: ["./src/test/setup.ts"],
  },
});

// Node 환경 - 순수 함수 테스트
export default defineConfig({
  test: {
    environment: "node", // DOM API 불필요
  },
});
```

## DOM Assertion 대안들

### 1. 순수 Vitest/Jest assertion

```typescript
// 기본 DOM 속성 검사
expect(element.tagName).toBe("BUTTON");
expect(element.className).toContain("active");
expect(element.textContent).toBe("Click me");
expect(element.getAttribute("disabled")).toBeNull();

// 장점: 의존성 없음, 명확한 검사
// 단점: 코드가 길어짐, 가독성 떨어짐
```

### 2. 커스텀 matcher 직접 구현

```typescript
// vitest.config.ts에서 확장
expect.extend({
  toBeVisible(received) {
    const pass = received.offsetParent !== null;
    return {
      message: () => `expected element to ${pass ? "not " : ""}be visible`,
      pass,
    };
  },
  toHaveTextContent(received, expected) {
    const pass = received.textContent === expected;
    return {
      message: () => `expected "${received.textContent}" to be "${expected}"`,
      pass,
    };
  },
});

// 사용
expect(element).toBeVisible();
expect(element).toHaveTextContent("Hello");

// 장점: 필요한 것만 구현, 프로젝트 특화
// 단점: 유지보수 부담, 테스트 코드 작성 필요
```

### 3. Chai + chai-dom

```bash
pnpm add -D chai chai-dom
```

```typescript
import { expect } from "chai";
import "chai-dom";

// 사용법이 jest-dom과 유사
expect(element).to.have.class("active");
expect(element).to.be.visible;
expect(element).to.contain.text("Hello");

// 장점: Jest 생태계 의존성 탈피
// 단점: Testing Library와 생태계 차이, 학습비용
```

### 4. 브라우저 기반 테스트

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    browser: {
      enabled: true,
      name: "chromium", // 실제 브라우저 사용
    },
  },
});

// 실제 브라우저 API 사용
const isVisible = await page.isVisible(".my-element");
const text = await page.textContent(".my-element");

// 장점: 완전한 브라우저 환경, 정확한 CSS 계산
// 단점: 느림, 복잡한 설정, CI 환경 구성 어려움
```

### 5. Playwright Component Testing

```bash
pnpm add -D @playwright/experimental-ct-react
```

```typescript
import { test, expect } from "@playwright/experimental-ct-react";

test("component test", async ({ mount }) => {
  const component = await mount(<MyButton>Click me</MyButton>);

  await expect(component).toHaveText("Click me");
  await expect(component).toBeVisible();

  // 장점: 실제 브라우저, 강력한 assertion
  // 단점: 설정 복잡, 느린 실행 속도
});
```

## Testing Library 프레임워크별 설정

### React 외 프레임워크

```bash
# Vue
pnpm add -D @testing-library/vue

# Angular
pnpm add -D @testing-library/angular

# Svelte
pnpm add -D @testing-library/svelte

# 바닐라 JS (DOM 조작 테스트)
pnpm add -D @testing-library/dom
```

## 성능 최적화 옵션

### Happy-DOM 사용 시 주의사항

```typescript
// 성능 최적화 설정
export default defineConfig({
  test: {
    environment: "happy-dom",
    // 병렬 처리 최적화
    pool: "threads",
    poolOptions: {
      threads: {
        maxThreads: 4,
        minThreads: 1,
      },
    },
  },
});
```

**검증해야 할 사항들**:

- CSS 스타일에 의존하는 컴포넌트 테스트
- Flexbox/Grid 레이아웃 테스트
- 복잡한 이벤트 처리 테스트
- 브라우저 API 의존성 체크

## 언제 대안을 고려할까?

### Happy-DOM 고려 상황

-  매우 간단한 컴포넌트만 테스트
-  성능이 절대적으로 중요한 CI 환경
-  CSS 스타일에 의존하지 않는 로직 테스트
-  빠른 프로토타이핑 단계

### 커스텀 matcher 고려 상황

-  특수한 DOM 검증 로직 필요
-  프로젝트 특화된 assertion 필요
-  성능이 매우 중요한 경우

### 브라우저 테스트 고려 상황

-  정확한 CSS 계산이 중요한 경우
-  복잡한 사용자 상호작용 테스트
-  실제 브라우저 동작 검증 필요

## 권장 마이그레이션 전략

### 1단계: 현재 설정 유지

기본 jsdom + jest-dom 조합으로 시작

### 2단계: 성능 측정

테스트 실행 시간이 문제가 되는지 확인

### 3단계: 점진적 도입

일부 간단한 테스트에만 Happy-DOM 적용

### 4단계: 검증 및 확장

문제없이 동작하는지 확인 후 점진적 확장

## 결론

대부분의 경우 [기본 가이드](./vitest.md)의 jsdom + jest-dom 조합이 최적입니다. 이 문서의 대안들은 다음과 같은 특수한 상황에서만 고려하세요:

- **성능이 매우 중요한 경우**: Happy-DOM 검토
- **특수한 assertion 필요**: 커스텀 matcher 구현
- **완벽한 브라우저 동작 필요**: 브라우저 테스트 도입

## 관련 문서

- [vitest.md](./vitest.md) - 기본 Vitest 설정 가이드
- [vitest-strategy.md](./vitest-strategy.md) - 테스트 전략 및 워크플로우
- [../quality/README.md](../quality/README.md) - 코드 품질 도구
- [../vite/vite-lib.md](../vite/vite-lib.md) - 라이브러리 빌드 설정
