# Prisma CLI + Azure Entra 인증 제한

Azure SQL에 Entra ID(구 Azure AD) 인증(`az login`)으로 Prisma를 쓸 때, **런타임과 CLI의 Entra 지원 여부가 다르다**.

---

## 표준 vs 현재 제한

### Prisma가 공식적으로 권장하는 것

| 항목 | 출처 | 내용 |
|------|------|------|
| **런타임** | [SQL Server docs](https://www.prisma.io/docs/orm/core-concepts/supported-databases/sql-server), [`@prisma/adapter-mssql`](https://www.npmjs.com/package/@prisma/adapter-mssql) | `PrismaClient({ adapter: new PrismaMssql(config) })` — **표준** |
| **Entra (런타임)** | adapter-mssql README | `authentication: { type: 'azure-active-directory-default' }` — **표준** |
| **CLI 연결** | [prisma.config.ts](https://www.prisma.io/docs/orm/reference/prisma-config-reference) | `datasource.url` + `user`/`password` JDBC 문자열 — **표준** |
| **스키마 변경 (장기)** | [Development and production](https://www.prisma.io/docs/orm/prisma-migrate/workflows/development-and-production) | `prisma migrate dev` → migration 파일 커밋 → `prisma migrate deploy` (CI) — **표준** |

### Prisma가 아직 공식 지원하지 않는 것

| 항목 | 출처 | 내용 |
|------|------|------|
| **CLI + Entra** | [GitHub #13853](https://github.com/prisma/prisma/issues/13853) | Rust CLI(`migrate`, `db push`, `db pull`) + Azure AD/Entra — **미지원** |
| **CLI + MSSQL adapter** | [GitHub #27822](https://github.com/prisma/prisma/issues/27822) | *"migrations with the MSSQL adapter aren't supported yet"* |
| **prisma.config `adapter`** | [Prisma config reference](https://www.prisma.io/docs/orm/reference/prisma-config-reference) | Prisma 7에서 **제거됨**. CLI용 adapter 설정 불가 |

---

## `prisma db push` — P1000 인증 실패

### 증상

```text
Error: P1000: Authentication failed against database server,
the provided database credentials for `` are not valid.
```

`DATABASE_URL`에 `authentication=ActiveDirectoryDefault`를 넣어도 동일.

### 원인

Prisma는 **두 경로**로 DB에 붙는다. Entra 지원 여부가 다르다.

| 용도 | 구현 | Entra (`az login`) |
|------|------|---------------------|
| 앱 런타임 · `db:test` · `db:seed` | `@prisma/adapter-mssql` + `azure-active-directory-default` | ✅ |
| CLI · `prisma db push`, `migrate`, `db pull` | Rust 스키마 엔진 + `prisma.config.ts`의 `datasource.url` | ❌ |

`DATABASE_URL`의 Entra 옵션은 **adapter-mssql** 연결 문자열 파서용이다. CLI Rust 엔진은 Entra 토큰을 획득하지 못해 사용자명이 빈 문자열(`for ```)로 SQL 인증을 시도하고 P1000이 난다.

`prisma.config.ts`의 `migrate.adapter`(Prisma 6 실험 기능)는 **Prisma 7에서 제거**되었다. adapter 설정만으로 `db push`를 대체할 수 없다.

### 해결 — Entra용 `db:push` 워크어라운드

커뮤니티·Prisma 이슈에서 널리 쓰이는 패턴:

1. `prisma migrate diff --from-empty --to-schema`로 SQL 생성 (DB 연결 불필요)
2. `mssql` + `azure-active-directory-default`로 SQL 실행 (런타임 adapter와 동일 인증)

```bash
# 1. 변경 SQL 생성 (DB 연결 불필요)
npx prisma migrate diff --from-empty --to-schema prisma/schema.prisma --script

# 2. 출력 SQL을 Entra 인증으로 실행
#    - Azure Data Studio (Microsoft Entra authentication), 또는
#    - adapter + mssql 실행 스크립트
```

---

## `db:seed` — PrismaClientInitializationError

Prisma 7 + SQL Server driver adapter 환경에서는 `PrismaClient` 생성 시 **반드시 `adapter`를 넘겨야** 한다. `DATABASE_URL`만으로는 초기화되지 않는다.

```ts
import { PrismaMssql } from '@prisma/adapter-mssql';
import { PrismaClient } from '@prisma/client';

const adapter = new PrismaMssql(buildMssqlConfig());
const prisma = new PrismaClient({ adapter });
```

`db:seed` 스크립트가 `.env`를 로드하지 않으면 `DB_SERVER` 등이 비어 adapter 설정도 실패할 수 있다. `--env-file=.env`를 명시한다.

---

## 자주 하는 오해

| 오해 | 실제 |
|------|------|
| `az login` 했는데 왜 안 되나? | 로그인 문제가 아니라 **Prisma CLI가 Entra를 지원하지 않음** |
| `DATABASE_URL`만 바꾸면 되나? | Rust CLI 경로에서는 Entra URL 형식이 동작하지 않음 |
| `prisma.config.ts`에 adapter 넣으면? | Prisma 7에서 제거됨. `datasource.url`은 여전히 필수 |
| `new PrismaClient()`만 쓰면 되나? | Prisma 7 SQL Server adapter 환경에서는 **`adapter` 필수** |

---

## 로컬 SQL Server (Entra 없음)

Docker / 로컬 SQL Server는 SQL 인증으로 CLI 전체 기능을 쓸 수 있다.

```env
DB_USE_ENTRA=false
DB_USER=sa
DB_PASSWORD=...
DATABASE_URL="sqlserver://localhost:1433;database=...;user=sa;password=...;encrypt=true;trustServerCertificate=true"
```

이 경우 표준 `prisma db push`를 호출한다.

## 관련 문서

- [환경 변수](./environment-variables.md) — 서버 사이드 env 관리
- [Prisma 공식 SQL Server 문서](https://www.prisma.io/docs/orm/core-concepts/supported-databases/sql-server)
