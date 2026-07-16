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

### `nest start --watch`(권장, Lunacast 검증됨)

dev 런타임을 Nest CLI tsc 컴파일러로 바꾼다.

**`package.json`**

```json
{
  "scripts": {
    "dev": "nest start --watch --env-file .env"
  },
  "devDependencies": {
    "@nestjs/cli": "^11.0.24"
  }
}
```

**`nest-cli.json`** (앱 루트, 표준 모드 — `monorepo: true` 아님)

```json
{
  "$schema": "https://json.schemastore.org/nest-cli",
  "collection": "@nestjs/schematics",
  "sourceRoot": "src",
  "compilerOptions": {
    "deleteOutDir": true
  }
}
```

**Lunacast에서 확인한 사항 (2026-07-17)**

- `tsconfig.build.json`이 없어도 `@nestjs/cli` v11은 `tsconfig.json`으로 폴백한다.
- `tsconfig.json`의 `references`(project references)와 충돌하지 않는다 — Nest CLI tsc는 참조 그래프를 무시하고 워크스페이스 `packages/*/dist`를 일반 모듈로 해석한다.
- Node 20.6+ / Nest CLI v11의 `--env-file .env`로 기존 `tsx watch --env-file=.env`와 동일하게 env를 로드한다.
- `build`/`start`(`tsc -b`, `node dist/main.js`)는 그대로 두고 **`dev` 스크립트만** 바꿔도 된다.
- DI가 필요한 엔드포인트가 `pnpm dev`에서 401 등 정상 응답을 반환함을 확인했다(500 `undefined` 재현 없음).

### 기타

- `tsx` 대신 `ts-node` + `swc` 등 데코레이터 메타데이터를 보존하는 러너를 쓴다
- 프로덕션과 동일하게 `build` 후 `node dist/main.js`로 검증한다
