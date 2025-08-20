# Node.js 개발 환경 설정 가이드

Node.js 프로젝트의 개발 환경 구성과 관련된 모든 설정 가이드를 포함합니다.

## 개요

Node.js 개발 환경은 다음과 같은 요소들로 구성됩니다:

- **버전 관리**: Node.js 버전 고정 및 팀 간 동기화
- **환경 변수**: 서버 사이드 환경 변수 관리 및 검증
- **스크립트**: 프로젝트 전반에 사용되는 공통 npm scripts

## 구성 요소

### 📦 [버전 관리](./version-management.md)

Node.js와 패키지 매니저 버전을 팀원 간에 동기화하는 방법:

- **.nvmrc**: NVM을 사용한 Node.js 버전 고정
- **.node-version**: nodenv/asdf 지원
- **package.json engines**: 최소 요구 버전 명시
- **Volta**: 차세대 Node.js 도구 체인 매니저

### 🌍 [환경 변수](./environment-variables.md)

서버 사이드 환경 변수 관리와 타입 안전성:

- **서버 환경 변수**: 데이터베이스, JWT, 외부 서비스 설정
- **TypeScript 타입 정의**: 환경 변수의 타입 안전성 보장
- **런타임 검증**: Zod를 사용한 환경 변수 유효성 검사
- **보안**: .gitignore 설정 및 민감 정보 관리

###  [스크립트 관리](./scripts.md)

프로젝트에서 자주 사용되는 npm scripts 패턴:

- **빌드 스크립트**: 개발/프로덕션 빌드 패턴
- **테스트 스크립트**: 단위/통합 테스트 실행
- **코드 품질**: 린트, 포맷팅, 타입 체크
- **환경별 실행**: 개발/스테이징/프로덕션 환경 분기

## 모노레포 연동

Node.js 환경 설정과 모노레포 워크스페이스 설정의 연관성:

- **패키지 매니저**: [monorepo/workspace.md](../monorepo/workspace.md)의 .npmrc/.pnpmrc 설정
- **워크스페이스**: pnpm workspace 및 Turbo 설정
- **의존성 관리**: 패키지 간 버전 동기화

## Vite 프로젝트와의 차이점

Node.js 일반 환경과 Vite 프로젝트의 환경 변수 관리 차이:

| 구분             | Node.js (server)           | Vite (client)                           |
| ---------------- | -------------------------- | --------------------------------------- |
| 환경 변수 접두사 | 제한 없음                  | `VITE_` 필요                            |
| 노출 범위        | 서버에서만                 | 클라이언트 번들에 포함                  |
| 보안 수준        | 높음 (민감 정보 포함 가능) | 낮음 (공개 정보만)                      |
| 설정 파일        | 이 가이드                  | [vite/vite-env.md](../vite/vite-env.md) |

## 시작하기

1. **버전 관리 설정**: [version-management.md](./version-management.md)로 시작
2. **환경 변수 구성**: [environment-variables.md](./environment-variables.md)에서 필요한 변수 정의
3. **스크립트 구성**: [scripts.md](./scripts.md)에서 프로젝트에 맞는 스크립트 선택
4. **모노레포 연동**: [monorepo/workspace.md](../monorepo/workspace.md)에서 워크스페이스 설정

## 참고 자료

- [Node.js 공식 문서](https://nodejs.org/docs/)
- [npm 스크립트 가이드](https://docs.npmjs.com/cli/v9/using-npm/scripts)
- [환경 변수 보안 가이드](https://12factor.net/config)
