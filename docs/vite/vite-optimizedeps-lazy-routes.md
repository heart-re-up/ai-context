# Vite 개발 서버 — 504 (Outdated Optimize Dep)

React Router Framework Mode 등 lazy 라우트를 쓰는 Vite 앱에서 콜드스타트 시 브라우저 콘솔에 아래와 같은 에러가 반복해서 뜨는 문제.

```text
GET http://localhost:3000/node_modules/.vite/deps/class-variance-authority.js?v=xxxxx net::ERR_ABORTED 504 (Outdated Optimize Dep)
GET http://localhost:3000/node_modules/.vite/deps/radix-ui.js?v=xxxxx net::ERR_ABORTED 504 (Outdated Optimize Dep)
...
```

터미널(vite 로그)에는 다음이 함께 보인다.

```text
✨ new dependencies optimized: <패키지 목록>
✨ optimized dependencies changed. reloading
```

---

## 근본 원인

Vite의 esbuild 기반 의존성 사전 번들링(`optimizeDeps`)은 **1차 스캔**에서 `index.html → entry.client.tsx`부터 **정적으로 도달 가능한 import**만 따라간다. React Router의 파일 기반 라우팅은 각 라우트를 `lazy()` **동적 import**로 분리하므로, 그 뒤에서만 쓰이는 의존성(예: UI 라이브러리가 재노출하는 `radix-ui`, `sonner`, `class-variance-authority` 등)은 1차 스캔에 걸리지 않는다.

사용자가 실제로 그 라우트를 열어야 Vite가 뒤늦게 새 의존성을 발견하고, 이때 **전체 `browserHash`가 재발급**된다(`node_modules/.vite/deps/_metadata.json`). `/node_modules/.vite/deps/*.js?v=<browserHash>` 형태의 URL은 전부 이 값을 공유하므로, 재발급 직전에 이미 나가있던 청크 요청(구버전 해시)이 `504 (Outdated Optimize Dep)`로 튕긴다. 이후 서버가 `full-reload`를 보내 브라우저가 자동으로 새로고침되긴 하지만, 그 사이 콘솔에는 에러가 남는다.

콜드스타트(캐시 무효화 — `vite.config.ts`/의존성/lockfile 변경, `.vite` 캐시 삭제 등)일 때만 재현되고, 캐시가 warm하게 재사용되는 일반 재시작에서는 발생하지 않는다.

### 왜 `@react-router/dev` Framework Mode에서만 두드러지나

라우트 파일을 **문자열 경로**로만 지정하면 `@react-router/dev/vite`(Framework Mode)는 빌드 시점에 이걸 라우트별 **동적 `import()`** (코드 스플리팅)로 자동 변환한다. esbuild 스캐너는 정적 import 그래프만 따라가고 `import()` 경계는 의도적으로 안 따라가므로, 라우트 파일 안의 의존성은 그 라우트를 실제로 열 때까지 스캐너 눈에 보이지 않는다.

반대로 App.tsx에서 각 라우트 컴포넌트를 **정적 import**해 `<Routes>`/`<Route element={...}>`로 직접 전개하는 방식(Declarative Mode에서 흔한 패턴)이라면 `entry → App.tsx → 라우트 컴포넌트 → 그 안의 의존성`까지 전부 정적 그래프라 1차 스캔이 한 번에 다 찾아내고, 이 문제 자체가 발생하지 않는다. 즉 원인은 React Router 자체가 아니라 **Framework Mode가 기본 제공하는 라우트 단위 자동 code-splitting**이다.

`optimizeDeps.entries`는 런타임 code-splitting(라우트별 청크 분리)은 그대로 유지한 채, **개발 서버의 사전 스캔 단계에서만** `import()` 뒤까지 강제로 훑게 하는 방식이라 Framework Mode를 유지하면서도 문제를 없앨 수 있다.

### 시행착오 — `optimizeDeps.include`로 개별 패키지 추가 (근본 해결 아님)

처음에는 콘솔에 찍힌 패키지들을 `optimizeDeps.include`에 하나씩 추가해 "1차 스캔에서 강제로 찾게" 했다. 이 방식은 **레이스 자체를 없애지 못한다** — 해당 패키지들은 안 걸리게 되지만, 스캔 대상이 늘어나 번들링 시간이 길어지면서(레이스 창 확대) 오히려 `react-dom/client`, `react-router/dom` 같은 더 핵심적인 의존성이 대신 504로 튀는 사례가 로그로 확인됐다. `include`는 라우트가 추가될 때마다 새로 걸릴 수 있는 두더지 잡기라 유지보수 부담도 크다.

## 해결

`optimizeDeps.entries`로 스캔 시작점을 `index.html` 하나가 아니라 **`src` 전체**로 넓혀서, `lazy()` 뒤의 라우트 파일까지 정적 파싱 대상에 포함시킨다. 이러면 최초 1회 스캔에서 앱 전체가 실제로 쓰는 의존성을 한꺼번에 찾아내 재최적화(=`browserHash` 재발급) 자체가 발생하지 않는다.

```ts
// vite.config.ts
optimizeDeps: {
  exclude: ['@ffmpeg/ffmpeg', '@ffmpeg/util'], // WASM 등 번들링 시 깨지는 패키지
  entries: ['src/**/*.{ts,tsx}'],
},
```

콜드스타트 시 서버가 처음 응답하기까지 스캔 범위가 넓어진 만큼 몇 초 더 걸릴 수 있다(정상).

### 검증 방법

`node_modules/.vite/deps/_metadata.json`의 `browserHash`를 확인한다. 서버를 켜고 여러 라우트를 돌아본 뒤에도 이 값이 **처음 값 그대로**여야 한다. 바뀌었다면 아직 스캔이 못 찾는 라우트가 남아있다는 뜻이다.

```bash
grep -E '"hash"|"browserHash"' node_modules/.vite/deps/_metadata.json
```

## `optimizeDeps` 옵션 요약

| 옵션 | 역할 |
|---|---|
| `entries` | esbuild 스캔의 **시작점**. 기본값은 `index.html`뿐 — lazy import 뒤는 못 봄. `src/**/*` 등으로 넓히면 동적 import 뒤의 의존성도 1차 스캔에 포함됨. |
| `include` | 자동 스캔이 못 찾는 특정 패키지를 수동으로 "미리 번들링" 강제 지정. 개별 보충용이라 근본 해결책은 아님. |
| `exclude` | 사전 번들링에서 제외. 이미 ESM이거나 번들링 시 깨지는 패키지(예: WASM을 동적 로드하는 `@ffmpeg/ffmpeg`)에 사용. |
| `force` | 캐시(`hash`/`configHash`/`lockfileHash`)를 무시하고 항상 재스캔. 디버깅용. |
| `holdUntilCrawlEnd` (기본 `true`) | `entries` 크롤링이 끝날 때까지 첫 응답을 지연시켜 재최적화 빈도를 줄임. `entries` 범위 밖(=발견 안 된 lazy 라우트)까지는 커버 못 함. |

## 관련 문서

- [개발 서버·HMR](./vite-dev.md) — Vite HMR 한계, 프록시 설정
