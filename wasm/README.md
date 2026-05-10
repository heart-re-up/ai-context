# WebAssembly (WASM) 개발 가이드

> 브라우저 환경에서 WASM을 사용할 때의 기본 개념, 제약, 설정 방법을 다룹니다.
> AI Agent는 이 문서를 기반으로 WASM 관련 코드를 작성하고 판단합니다.

---

## WASM이란

WebAssembly는 C/C++/Rust 등으로 작성된 코드를 브라우저에서 실행할 수 있도록 컴파일한 바이너리 포맷입니다. JavaScript보다 낮은 수준에서 실행되며, CPU 집약적인 연산(영상 처리, 암호화 등)에 적합합니다.

브라우저에서 WASM은 JavaScript와 동일한 **메인 스레드** 또는 **Web Worker** 위에서 실행됩니다. WASM 자체가 멀티스레드를 제공하지는 않으며, 스레드가 필요하다면 Web Worker와 조합해야 합니다.

---

## 핵심 제약 사항

### 1. 싱글스레드 WASM vs 멀티스레드 WASM

| 구분 | 설명 | SharedArrayBuffer 필요 |
|---|---|---|
| 싱글스레드 | 메인 스레드 또는 Worker 하나에서만 실행 | 불필요 |
| 멀티스레드 | pthread 기반, Worker 간 메모리 공유 | **필수** |

멀티스레드 빌드를 사용하는 라이브러리(예: `@ffmpeg/ffmpeg` 멀티스레드 코어)는 반드시 `SharedArrayBuffer`가 활성화된 환경에서 동작합니다.

싱글스레드 빌드를 제공하는 경우 `SharedArrayBuffer` 없이도 사용 가능하지만, 성능이 낮습니다.

### 2. SharedArrayBuffer 활성화 조건

`SharedArrayBuffer`는 보안상 이유(Spectre 공격 대응)로 **Cross-Origin Isolation** 상태에서만 사용 가능합니다. 이를 위해 다음 두 HTTP 응답 헤더가 **반드시** 필요합니다.

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

이 헤더가 없으면 `SharedArrayBuffer is not defined` 런타임 오류가 발생합니다.

> ⚠️ **부작용**: 이 헤더가 활성화되면 `window.opener`가 `null`이 되고,
> 다른 출처의 리소스(`<img>`, `<script>` 등)에 `crossorigin` 속성이 없으면
> 로딩이 차단됩니다. 서드파티 스크립트(광고, 소셜 로그인 팝업 등)와 충돌할 수 있습니다.

### 3. 브라우저 지원 현황

| 기능 | Chrome | Firefox | Safari | Edge |
|---|---|---|---|---|
| WASM 기본 | ✅ 57+ | ✅ 52+ | ✅ 11+ | ✅ 16+ |
| SharedArrayBuffer | ✅ 92+ | ✅ 79+ | ✅ 15.2+ | ✅ 92+ |
| WASM SIMD | ✅ 91+ | ✅ 89+ | ✅ 16.4+ | ✅ 91+ |
| WASM Threads | ✅ 74+ | ✅ 79+ | ✅ 14.1+ | ✅ 79+ |

모바일 Safari는 `SharedArrayBuffer`를 15.2부터 지원하므로, iOS 15.2 미만은 멀티스레드 WASM을 사용할 수 없습니다.

### 4. 메모리 제한

WASM 모듈은 선형 메모리(Linear Memory)를 사용합니다. 브라우저별 최대 할당량이 다르며, 32비트 WASM의 경우 이론상 최대 4GB이지만 실제로는 브라우저와 OS 환경에 따라 제한됩니다. 대용량 파일을 처리할 때는 청크 단위 처리나 메모리 해제(`deleteFile`, `free` 등)를 반드시 수행해야 합니다.

---

## Vite 프로젝트 공통 설정

### vite.config.ts

```typescript
export default defineConfig({
  server: {
    headers: {
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
  preview: {
    headers: {
      "Cross-Origin-Opener-Policy": "same-origin",
      "Cross-Origin-Embedder-Policy": "require-corp",
    },
  },
  optimizeDeps: {
    // WASM을 포함하는 패키지는 Vite의 esbuild 사전 번들링에서 반드시 제외
    // 포함 시 WASM 바이너리가 손상되거나 런타임 오류 발생
    exclude: ["<wasm-package-name>"],
  },
});
```

### 배포 환경 헤더 설정

Vite 개발 서버의 `headers` 설정은 개발 환경에서만 유효합니다. 배포 환경에서는 호스팅 서비스별로 별도 설정이 필요합니다.

#### Azure Static Web Apps (권장)

`staticwebapp.config.json`을 프로젝트 루트에 두면 모든 응답에 헤더가 자동 적용됩니다.

```json
{
  "globalHeaders": {
    "Cross-Origin-Opener-Policy": "same-origin",
    "Cross-Origin-Embedder-Policy": "require-corp"
  },
  "navigationFallback": {
    "rewrite": "/index.html"
  }
}
```

> Azure Blob Storage의 **Static Website** 기능(`$web` 컨테이너)은 응답 헤더 커스터마이징을 지원하지 않습니다. COOP/COEP 헤더가 필요한 WASM 프로젝트는 반드시 **Azure Static Web Apps**를 사용해야 합니다. `$web` Static Website에 Azure CDN/Front Door를 붙여 헤더 규칙을 추가하는 방법도 있지만 구성이 복잡하고 비용이 발생합니다.

#### Vercel (`vercel.json`)

```json
{
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        { "key": "Cross-Origin-Opener-Policy", "value": "same-origin" },
        { "key": "Cross-Origin-Embedder-Policy", "value": "require-corp" }
      ]
    }
  ]
}
```

#### Netlify (`netlify.toml`)

```toml
[[headers]]
  for = "/*"
  [headers.values]
    Cross-Origin-Opener-Policy = "same-origin"
    Cross-Origin-Embedder-Policy = "require-corp"
```

#### GitHub Pages (`public/_headers`)

```
/*
  Cross-Origin-Opener-Policy: same-origin
  Cross-Origin-Embedder-Policy: require-corp
```

---

## WASM 로딩 패턴

### 기본 원칙

- WASM 바이너리는 크기가 크므로(수 MB~수십 MB) **지연 로딩(Lazy Loading)** 을 원칙으로 합니다.
- 초기 페이지 진입 시 로드하지 않고, 사용자가 실제로 기능을 사용하려는 시점에 로드합니다.
- 로딩 상태(`idle` | `loading` | `ready` | `error`)를 명확히 관리합니다.

### 로딩 상태 타입 패턴

```typescript
type WasmLoadState = "idle" | "loading" | "ready" | "error";
```

### Web Worker 분리

WASM 연산이 메인 스레드에서 실행되면 UI가 블로킹됩니다. 연산 시간이 길 경우(수 초 이상) Web Worker로 분리하는 것을 권장합니다.

```
메인 스레드  ──[postMessage]──▶  Worker
             ◀──[postMessage]──  Worker (진행률, 결과)
```

Worker 내부에서 WASM을 로드하고 실행하면 메인 스레드는 UI 반응성을 유지할 수 있습니다.

---

## 자주 발생하는 오류

| 오류 메시지 | 원인 | 해결 방법 |
|---|---|---|
| `SharedArrayBuffer is not defined` | COOP/COEP 헤더 누락 | `vite.config.ts`에 헤더 설정 추가 |
| `Failed to fetch` (WASM 로딩 실패) | CORS 또는 헤더 문제 | 배포 환경 헤더 설정 확인 |
| WASM 모듈이 `optimizeDeps`에 포함됨 | Vite가 WASM을 esbuild로 번들링 시도 | `optimizeDeps.exclude`에 패키지 추가 |
| `out of memory` | WASM 선형 메모리 초과 | 처리 후 메모리 명시적 해제 |
| Worker에서 WASM 로딩 실패 | Worker 컨텍스트에서 모듈 경로 불일치 | `import.meta.url` 기반 경로 사용 |

---

## 참고 자료

- [MDN — WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)
- [MDN — SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer)
- [MDN — Cross-Origin Isolation](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now#security_requirements)
- [web.dev — Cross-Origin Isolation 가이드](https://web.dev/cross-origin-isolation-guide/)

## 관련 문서

- [`../vite/vite.md`](../vite/vite.md) — COOP/COEP 배포 환경 헤더 설정
- [`../vite/vite-dev.md`](../vite/vite-dev.md) — Vite 개발 서버 WASM 설정
