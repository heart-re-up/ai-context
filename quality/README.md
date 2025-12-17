# 코드 품질 관리 가이드

코드 품질을 유지하기 위한 도구 설정과 사용법을 다룹니다.

## 목차

- [Prettier](./prettier.md) - 코드 포맷팅 도구 설정
- [ESLint](./eslint.md) - 코드 린팅 및 품질 규칙 설정
- [EditorConfig](./editorconfig.md) - 에디터 간 일관성 유지 설정
- [스크립트 및 자동화](./scripts.md) - 코드 품질 관련 스크립트와 CI/CD 통합
- [모노레포 설정 구현](./monorepo-config.md) - 모노레포용 설정 패키지 실제 구현

## 개요

이 디렉토리는 코드 품질을 유지하기 위한 도구들의 **상세한 설정 방법과 실제 구현**을 다룹니다.

>  **모노레포에서의 설정 구조화**는 [`monorepo/config.md`](../monorepo/config.md)를 참고하세요.

### 주요 도구

1. **Prettier**: 코드 포맷팅 자동화
2. **ESLint**: 코드 품질 검사 및 오류 탐지
3. **EditorConfig**: 개발 환경 간 일관성 유지
4. **Git Hooks**: 커밋 전 자동 검사

### 권장 워크플로우

1. 개발 중: 에디터에서 실시간 ESLint 검사
2. 저장 시: Prettier 자동 포맷팅
3. 커밋 전: lint-staged로 변경된 파일만 검사
4. CI/CD: 전체 코드베이스 품질 검증

## 빠른 시작

```bash
# 의존성 설치
pnpm add -D prettier eslint typescript-eslint

# 설정 파일 생성 (각 도구별 가이드 참조)
# - prettier.config.mjs
# - eslint.config.mjs
# - .editorconfig

# 코드 검사 및 포맷팅
pnpm lint
pnpm format
```

## 도구별 특징

| 도구              | 목적        | 설정 파일             | 주요 기능            | 상세 가이드                                |
| ----------------- | ----------- | --------------------- | -------------------- | ------------------------------------------ |
| Prettier          | 코드 포맷팅 | `prettier.config.mjs` | 일관된 코드 스타일   | [prettier.md](./prettier.md)               |
| ESLint            | 코드 품질   | `eslint.config.mjs`   | 오류 탐지, 코딩 규칙 | [eslint.md](./eslint.md)                   |
| EditorConfig      | 에디터 설정 | `.editorconfig`       | 에디터 간 일관성     | [editorconfig.md](./editorconfig.md)       |
| Husky/lint-staged | Git 훅      | `package.json`        | 커밋 전 자동 검사    | [scripts.md](./scripts.md)                 |
| 모노레포 설정     | 설정 패키지 | `config/*/`           | 중앙화된 설정 관리   | [monorepo-config.md](./monorepo-config.md) |

## 설정 접근 방법

###  구조화 (Monorepo)

모노레포에서 설정을 패키지로 관리하는 방법

- 설정 패키지 구성
- 모듈간 참조 방법
- 워크스페이스 통합

 **[monorepo/config.md](../monorepo/config.md)**

###  실제 구현 (Quality)

각 도구의 상세한 설정 파일 작성 방법

- 구체적인 설정 옵션
- 실전 예시와 팁
- 문제 해결 방법

 **이 디렉토리의 각 문서들**
