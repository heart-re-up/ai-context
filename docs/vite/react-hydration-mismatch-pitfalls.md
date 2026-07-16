# React 하이드레이션 오류(#418) — 흔한 함정

React error #418(치명적 하이드레이션 불일치)이 발생하는 두 가지 대표 원인과 해결.

---

## 1. `prerender` + SPA fallback 설정 불일치

### 증상

- 홈(`/`)을 새로고침하면 정상이지만, 다른 페이지를 새로고침하면 React error #418이 발생한다.
- 콘솔에 `Error: No route matches URL "/..."` 같은 React Router 404 에러 바운더리 메시지가 같이 뜨기도 한다.

### 원인

React Router v7에서 `ssr: false` + `prerender: ["/"]`로 설정하면 **두 개의 서로 다른 정적 HTML**이 만들어진다.

- `dist/client/index.html` — `/` 경로만을 위해 **완전히 렌더링된** 홈페이지 HTML (loader 데이터·실제 DOM 구조 포함).
- `dist/client/__spa-fallback.html` — `/` 외 **다른 모든 경로**를 위한 범용 SPA 셸(빈 로딩 화면).

정적 호스팅의 `navigationFallback.rewrite`가 `/index.html`로 되어 있으면, Azure Static Web Apps·Netlify·Vercel 등이 `/dashboard` 같은 경로 요청도 전부 **홈페이지가 그대로 렌더링된 `index.html`**로 응답한다. 브라우저 URL은 `/dashboard`인데 받은 HTML은 홈페이지 DOM이므로, `hydrateRoot(document, ...)`가 시도하는 하이드레이션이 구조적으로 전혀 다른 두 트리를 비교하게 되어 React error #418이 발생한다.

이는 React Router 공식 문서에도 명시된 동작이다: [Pre-Rendering with a SPA Fallback](https://reactrouter.com/how-to/pre-rendering#pre-rendering-with-a-spa-fallback). `/`를 `prerender`에 포함하면 `index.html`은 `/` 전용이 되고, 다른 경로는 반드시 `__spa-fallback.html`로 서빙해야 한다.

`prerender` 옵션이 없는 "순수 SPA 모드"라면 `index.html` 자체가 (빈 로딩 셸이라) 모든 경로에 대해 하이드레이션 가능하므로 이 문제가 발생하지 않는다. `prerender: ["/"]`를 쓰는 앱에서만 해당된다.

### 해결

정적 호스팅 rewrite 설정에서 fallback 대상을 `index.html` → `__spa-fallback.html`로 변경한다.

```json
// Azure SWA 예시 (staticwebapp.config.json)
{
  "navigationFallback": {
    "rewrite": "/__spa-fallback.html"
  }
}
```

Netlify `_redirects`, Vercel `rewrites` 등도 동일 원리 — `prerender`된 홈 전용 HTML과 범용 SPA 셸을 구분해 서빙해야 한다.

CSP를 `enforce` 모드로 패치할 때 `index.html`과 `__spa-fallback.html`의 인라인 `<script>` 해시가 서로 다르므로, 두 파일 모두 해시 계산 대상에 포함해야 한다.

### 재현/검증

1. `pnpm build` 후 `dist/client`를 정적 서버로 서빙
2. fallback을 `index.html`로 서빙 → `/dashboard` 직접 접속 시 React error #418 재현
3. fallback을 `__spa-fallback.html`로 서빙 → 동일 경로에서 오류 없이 정상 렌더링

---

## 2. 서드파티 스크립트를 하이드레이션 커밋 전에 주입

### 증상

- 루트 페이지든 다른 페이지든 새로고침 시 React error #418이 발생한다.
- Analytics/Clarity/App Insights 등 텔레메트리를 켠 뒤부터 발생하기 시작했다.

### 원인

`initTelemetry()`(Clarity/App Insights 스크립트를 `document`에 동적으로 삽입)를 `entry.client.tsx`의 모듈 최상단(즉 `hydrateRoot(document, ...)` 호출 **이전**)에서 실행하면, `hydrateRoot`가 하이드레이션하려는 DOM(`<head>`/`<body>`)을 telemetry가 먼저 변형시켜 React 하이드레이션 불일치 오류(#418)가 발생한다. Clarity npm 패키지의 `injectScript`는 `document.getElementsByTagName('script')[0]` 기준으로 동기적으로 `<script>`를 삽입하므로, 하이드레이션 전에 실행되면 서버 렌더 결과와 클라이언트 DOM이 어긋난다.

### 해결

Analytics/Clarity 등 서드파티 스크립트 초기화는 **React 커밋(하이드레이션 포함) 이후**에만 실행한다.

```tsx
// ❌ 모듈 최상단에서 즉시 실행
initTelemetry();

// ✅ 마운트 useEffect 안에서 실행
function App() {
  useEffect(() => {
    initTelemetry();
  }, []);
  // ...
}
```

또는 전용 `TelemetryPageTracker` 컴포넌트가 `initOptions` prop을 받아 **마운트 `useEffect` 안에서** `initTelemetry()`를 호출하게 한다. React는 effect를 커밋 이후에만 실행하므로, telemetry로 인한 DOM 변형이 하이드레이션과 절대 겹치지 않는다.

## 관련 문서

- [개발 서버·HMR](./vite-dev.md)
- [환경 변수](./vite-env.md)
