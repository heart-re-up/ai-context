# 빌드 설정 가이드

프로젝트의 빌드 도구와 설정 방법을 다루는 종합 가이드입니다.

## 개요

현대적인 프론트엔드 프로젝트에서 빌드 시스템은 다음과 같은 역할을 담당합니다:

- **개발 환경**: 빠른 개발 서버, HMR, 디버깅 도구
- **프로덕션 빌드**: 최적화된 번들, 압축, 트리 셰이킹
- **라이브러리 빌드**: 재사용 가능한 모듈 패키징

## 구성 요소

### 📂 [Vite 기본 설정](./vite.md)

Vite의 기본 설정과 핵심 개념을 다룹니다:

- **기본 설정 구조**: 프로젝트 초기 설정
- **경로 별칭 (Path Alias)**: 모듈 경로 단축 설정
- **조건부 설정**: 환경별 설정 분기
- **성능 최적화**: 빌드 속도와 번들 크기 최적화

### 🔌 [Vite 플러그인 가이드](./vite-plugins.md)

Vite 생태계의 다양한 플러그인들과 설정 방법:

- **React 관련**: @vitejs/plugin-react, @vitejs/plugin-react-swc
- **정적 파일 처리**: vite-plugin-static-copy
- **SVG 처리**: vite-plugin-svgr, vite-plugin-svg-icons
- **번들 분석**: rollup-plugin-visualizer, vite-bundle-analyzer
- **개발 도구**: vite-plugin-eslint, vite-plugin-checker
- **PWA 및 최적화**: vite-plugin-pwa, @vitejs/plugin-legacy
- **커스텀 플러그인 개발**: 플러그인 제작 가이드

###  [개발서버 설정](./vite-dev.md)

React 개발 환경 구성과 개발서버 최적화:

- **HMR 설정**: Hot Module Replacement 최적화
- **프록시 설정**: 백엔드 API 연동
- **디버깅 도구**: 소스맵, React DevTools 설정
- **성능 최적화**: 개발 빌드 속도 향상

### 🌍 [환경 변수 설정](./vite-env.md)

Vite 프로젝트의 환경 변수 관리:

- **환경 파일 구성**: .env.development, .env.production 등
- **타입 안전성**: TypeScript 환경 변수 타입 정의
- **유효성 검사**: Zod를 사용한 런타임 검증
- **보안**: 민감한 정보 처리 방법
- **모범 사례**: 환경별 설정 관리

### 📦 [라이브러리 빌드](./vite-lib.md)

모듈 패키징과 라이브러리 배포:

- **기본 라이브러리 설정**: 단일/다중 진입점 설정
- **TypeScript 지원**: 타입 선언 파일 생성
- **React 컴포넌트 라이브러리**: CSS 처리, 외부 의존성 관리
- **배포 설정**: npm 패키지 배포 준비
- **트리 셰이킹**: 번들 크기 최적화

### 🧪 [테스트 설정](../vitest/README.md)

Vitest를 사용한 라이브러리 테스트 환경:

- **기본 테스트 설정**: React 컴포넌트, 훅, 유틸리티 테스트
- **커버리지 설정**: 코드 커버리지 측정 및 임계값 설정
- **CI/CD 통합**: GitHub Actions와의 연동
- **성능 최적화**: 테스트 실행 속도 향상
- **모범 사례**: 효과적인 테스트 작성 패턴

## 빌드 시나리오별 가이드

### 📱 애플리케이션 빌드

**SPA (Single Page Application)**

```bash
# 개발 서버 시작
npm run dev

# 프로덕션 빌드
npm run build

# 빌드 미리보기
npm run preview
```

**설정 파일**: `vite.config.ts`

### 📚 라이브러리 빌드

**React 컴포넌트 라이브러리**

```bash
# 개발 중 빌드 감시
npm run build:watch

# 라이브러리 빌드
npm run build:lib

# 패키지 배포
npm publish
```

**설정 파일**: `vite.lib.config.ts`

### 🏢 모노레포 빌드

**워크스페이스별 빌드**

```bash
# 특정 워크스페이스 빌드
pnpm --filter=@project/ui-component build

# 모든 워크스페이스 빌드
pnpm -r build
```

## 설정 파일 구조

### 기본 구조

```
project/
├── vite.config.ts          # 기본 애플리케이션 설정
├── vite.lib.config.ts      # 라이브러리 빌드 설정 (선택)
├── tsconfig.json           # TypeScript 설정
├── tsconfig.build.json     # 빌드용 TypeScript 설정
└── package.json            # 빌드 스크립트 정의
```

### 모노레포 구조

```
monorepo/
├── apps/
│   ├── main-app/
│   │   ├── vite.config.ts
│   │   └── tsconfig.json
│   └── admin-app/
│       ├── vite.config.ts
│       └── tsconfig.json
├── packages/
│   ├── ui-component/
│   │   ├── vite.config.ts  # 라이브러리 빌드
│   │   └── tsconfig.json
│   └── shared/
│       ├── vite.lib.config.ts
│       └── tsconfig.build.json
└── package.json            # 루트 빌드 스크립트
```

## 환경별 설정

### 개발 환경

- **빠른 시작**: ESBuild 기반 빠른 번들링
- **HMR**: 실시간 코드 변경 반영
- **소스맵**: 디버깅을 위한 원본 코드 매핑
- **프록시**: API 서버 연동

### 스테이징 환경

- **부분 최적화**: 개발과 프로덕션의 중간 설정
- **디버깅 가능**: 소스맵 유지
- **성능 테스트**: 실제 번들 크기 확인

### 프로덕션 환경

- **최적화**: 코드 압축, 트리 셰이킹
- **캐싱**: 장기 캐싱을 위한 해시 파일명
- **번들 분석**: 크기 최적화 확인

## 성능 최적화 전략

### 빌드 속도 최적화

1. **의존성 사전 번들링**

   ```typescript
   optimizeDeps: {
     include: ['react', 'react-dom'],
   }
   ```

2. **멀티코어 활용**

   ```typescript
   build: {
     minify: 'esbuild', // 빠른 압축
   }
   ```

3. **캐시 활용**
   ```bash
   # 의존성 캐시 활용
   node_modules/.vite/
   ```

### 번들 크기 최적화

1. **코드 분할**

   ```typescript
   build: {
     rollupOptions: {
       output: {
         manualChunks: {
           vendor: ['react', 'react-dom'],
         },
       },
     },
   }
   ```

2. **트리 셰이킹**

   ```json
   {
     "sideEffects": false
   }
   ```

3. **동적 임포트**
   ```typescript
   const Component = lazy(() => import("./Component"));
   ```

## 문제 해결

### 자주 발생하는 이슈

1. **빌드 실패**

   - TypeScript 에러 확인
   - 의존성 버전 충돌 해결
   - 메모리 부족 시 Node.js 메모리 증가

2. **번들 크기 문제**

   - 번들 분석기로 큰 의존성 확인
   - 불필요한 의존성 제거
   - 동적 임포트 적용

3. **HMR 작동 안함**
   - 파일 확장자 확인
   - Fast Refresh 설정 확인
   - 프록시 설정 점검

### 디버깅 도구

```bash
# 상세 빌드 로그
DEBUG=vite:* npm run build

# 번들 분석
npm run build -- --report

# 메모리 사용량 확인
NODE_OPTIONS="--inspect" npm run build
```

## 배포 전략

### 정적 호스팅

```typescript
// GitHub Pages, Netlify, Vercel
export default defineConfig({
  base: "/repository-name/", // 베이스 경로 설정
  build: {
    outDir: "dist",
  },
});
```

### Docker 배포

```dockerfile
# 멀티스테이지 빌드
FROM node:18-alpine as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
```

### CDN 배포

```typescript
export default defineConfig({
  build: {
    rollupOptions: {
      external: ["react", "react-dom"], // CDN에서 로드
    },
  },
});
```

## 모범 사례

### 설정 관리

1. **환경별 분리**: 개발/스테이징/프로덕션 설정 분리
2. **공통 설정**: 재사용 가능한 설정 모듈화
3. **타입 안전성**: TypeScript로 설정 파일 작성
4. **문서화**: 설정 변경 시 문서 업데이트

### 성능 모니터링

1. **빌드 시간**: 정기적인 빌드 시간 측정
2. **번들 크기**: 크기 증가 추이 모니터링
3. **의존성 관리**: 불필요한 의존성 정기 검토
4. **캐시 효율성**: 캐시 적중률 확인

### 팀 협업

1. **설정 공유**: .vscode, .idea 설정 공유
2. **스크립트 통일**: 동일한 빌드 스크립트 사용
3. **환경 동기화**: 동일한 Node.js 버전 사용
4. **문제 공유**: 빌드 이슈 해결 방법 문서화

## 도구 비교

### Vite vs Webpack

| 특징            | Vite                | Webpack          |
| --------------- | ------------------- | ---------------- |
| 개발 서버       | ESBuild (매우 빠름) | 느림             |
| 설정 복잡도     | 낮음                | 높음             |
| 플러그인 생태계 | 성장 중             | 성숙함           |
| 번들 크기       | 최적화됨            | 설정에 따라 다름 |

### Vite vs Rollup

| 특징            | Vite           | Rollup         |
| --------------- | -------------- | -------------- |
| 개발 환경       | 개발 서버 내장 | 별도 설정 필요 |
| 라이브러리 빌드 | 간편한 설정    | 더 세밀한 제어 |
| 학습 곡선       | 완만함         | 가파름         |

## 참고 자료

- [Vite 공식 문서](https://vitejs.dev/)
- [Rollup 공식 문서](https://rollupjs.org/)
- [ESBuild 공식 문서](https://esbuild.github.io/)
- [TypeScript 컴파일러 옵션](https://www.typescriptlang.org/tsconfig)

## 관련 문서

- [../typescript.md](../typescript.md) - TypeScript 설정
- [../quality.md](../quality.md) - 코드 품질 도구
- [../vitest/README.md](../vitest/README.md) - Vitest 테스트 설정
