# Vitest 테스트 가이드

Vitest를 사용한 라이브러리 테스트 환경 구성과 전략에 대한 종합 가이드입니다.

## 📁 문서 구성

### 🚀 [vitest.md](./vitest.md) - 기본 설정 가이드

Vitest 설치부터 기본 설정까지, 테스트 환경 구축의 모든 것

**다루는 내용:**

- Vitest 기본 설치 및 설정
- jsdom 환경 구성
- Testing Library 통합
- 커버리지 설정
- CI/CD 통합

**이런 분에게 추천:**

- Vitest를 처음 사용하는 경우
- 기본적인 테스트 환경이 필요한 경우
- 라이브러리 개발용 설정을 찾는 경우

---

### 📋 [vitest-strategy.md](./vitest-strategy.md) - 테스트 전략

실용적인 테스트 전략과 개발 워크플로우

**다루는 내용:**

- 단위 테스트 vs 통합 테스트 전략
- 프로젝트 구조 설계
- 개발 워크플로우 최적화
- 품질 관리 방법
- 모범 사례

**이런 분에게 추천:**

- 체계적인 테스트 전략이 필요한 경우
- 라이브러리 + 데모 앱 구조를 고려하는 경우
- 팀 차원의 테스트 가이드라인이 필요한 경우

---

### ⚡ [vitest-options.md](./vitest-options.md) - 고급 옵션

성능 최적화와 대안 환경들

**다루는 내용:**

- Happy-DOM vs jsdom 비교
- 커스텀 matcher 구현
- 브라우저 테스트 옵션
- 성능 최적화 방법
- 대안 도구들

**이런 분에게 추천:**

- 성능 최적화가 필요한 경우
- 특수한 테스트 요구사항이 있는 경우
- 대안 도구들을 검토하고 싶은 경우

## 🎯 시작하기 권장 순서

### 1. 처음 시작하는 경우

1. **[vitest.md](./vitest.md)** 읽고 기본 환경 구축
2. **[vitest-strategy.md](./vitest-strategy.md)** 읽고 프로젝트 구조 설계
3. 필요시 **[vitest-options.md](./vitest-options.md)** 참고

### 2. 기존 환경 개선하는 경우

1. **[vitest-strategy.md](./vitest-strategy.md)** 읽고 현재 전략 검토
2. **[vitest-options.md](./vitest-options.md)** 읽고 최적화 방안 검토
3. **[vitest.md](./vitest.md)** 참고하여 설정 개선

## 🔗 연관 문서

- [../quality/README.md](../quality/README.md) - 코드 품질 도구 (ESLint, Prettier 등)
- [../vite/README.md](../vite/README.md) - Vite 빌드 설정
- [../monorepo/README.md](../monorepo/README.md) - 모노레포 구성

## 📊 문서별 특징 비교

| 문서                   | 난이도 | 주요 초점         | 분량   |
| ---------------------- | ------ | ----------------- | ------ |
| **vitest.md**          | 초급   | 기술적 설정       | 상세함 |
| **vitest-strategy.md** | 중급   | 전략과 워크플로우 | 포괄적 |
| **vitest-options.md**  | 고급   | 최적화와 대안     | 전문적 |

## 💡 빠른 참조

### 자주 찾는 설정

```typescript
// 기본 vitest.config.ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",
    setupFiles: ["./tests/setup.ts"],
    globals: true,
  },
});
```

### 자주 사용하는 명령어

```bash
# 테스트 실행
pnpm test

# 커버리지 확인
pnpm test:coverage

# UI 모드
pnpm test:ui
```

---

**💡 TIP**: 각 문서는 독립적으로 읽을 수 있지만, 전체적인 이해를 위해서는 순서대로 읽는 것을 권장합니다.
