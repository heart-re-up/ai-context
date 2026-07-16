# NestJS API — `tsx watch` dev 런타임 함정

`apps/*-api`의 `dev` 명령이 `tsx watch`(esbuild)를 쓰는 경우, **esbuild는 `emitDecoratorMetadata`를 지원하지 않는다** ([esbuild#257](https://github.com/evanw/esbuild/issues/257)). 그 결과 NestJS 타입 기반 생성자 DI가 깨져 서비스 주입 필드가 `undefined`가 되고, DB/서비스를 쓰는 엔드포인트는 500 `Cannot read properties of undefined (...)`를 반환한다. **코드 버그가 아니라 러너 한계**다.

## `pnpm dev`에서 기대되는 동작

- API 서버 부팅은 정상
- 주입이 없는 엔드포인트(`GET /health`)는 정상

## DB·서비스 연동 엔드포인트 검증

**tsc로 컴파일된 산출물**로 실행한다(데코레이터 메타데이터 보존):

```bash
pnpm --filter @scope/api build
node apps/api/dist/main.js
```

각 API 패키지에 동일하게 적용한다.

## 대안

- dev 런타임을 `nest start --watch`(tsc 기반)로 바꾸거나
- `tsx` 대신 `ts-node` + `swc` 등 데코레이터 메타데이터를 보존하는 러너를 쓴다
- 프로덕션과 동일하게 `build` 후 `node dist/main.js`로 검증한다
