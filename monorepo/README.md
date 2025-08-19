# Monorepo 설정 가이드

이 디렉토리는 모노레포(Monorepo) 구조와 관련된 모든 설정 가이드를 포함합니다.

## 개요

모노레포는 여러 개의 프로젝트나 패키지를 하나의 저장소에서 관리하는 개발 방식입니다.
이 구조는 코드 공유, 일관된 도구 사용, 그리고 프로젝트 간 의존성 관리를 효율적으로 할 수 있게 해줍니다.

## 주요 구성 요소

### 1. 패키지 매니저 및 워크스페이스

- **pnpm workspace**: 효율적인 패키지 관리
- **Turbo**: 빌드 및 태스크 오케스트레이션
- 자세한 내용: [workspace.md](./workspace.md)

### 2. 애플리케이션 디렉토리 (apps/)

- 실제 배포되는 애플리케이션들
- 웹 앱, 모바일 앱, 서버 애플리케이션 등
- 자세한 내용: [apps.md](./apps.md)

### 3. 패키지 디렉토리 (packages/)

- 재사용 가능한 라이브러리 및 유틸리티
- 공유 컴포넌트, 유틸리티 함수, 타입 정의 등
- 자세한 내용: [packages.md](./packages.md)

### 4. 설정 관리 (config/)

- ESLint, TypeScript, Prettier 등 공유 설정
- 프로젝트 전체의 일관된 코드 품질 유지
- 자세한 내용: [config.md](./config.md)

## 모노레포의 장점

### 코드 공유

- 공통 로직과 컴포넌트를 쉽게 공유
- 중복 코드 최소화
- 일관된 코딩 스타일 유지

### 의존성 관리

- 모든 프로젝트의 의존성을 중앙에서 관리
- 버전 호환성 문제 해결
- 보안 업데이트의 일관성

### 개발 효율성

- 하나의 저장소에서 모든 프로젝트 관리
- 통합된 CI/CD 파이프라인
- 크로스 프로젝트 리팩토링 용이

## 기본 구조

```
project-root/
├── apps/                    # 애플리케이션들
│   ├── web-app/            # 웹 애플리케이션
│   ├── mobile-app/         # 모바일 애플리케이션
│   └── admin-dashboard/    # 관리자 대시보드
├── packages/               # 공유 패키지들
│   ├── ui-components/      # UI 컴포넌트 라이브러리
│   ├── shared-lib/         # 공유 유틸리티 및 로직
│   └── types/              # 공유 타입 정의
├── config/                 # 공유 설정 (권장)
│   ├── eslint-config/      # ESLint 설정
│   ├── typescript-config/  # TypeScript 설정
│   └── prettier-config/    # Prettier 설정
├── package.json            # 루트 패키지 설정
├── pnpm-workspace.yaml     # 워크스페이스 설정
├── turbo.json              # Turbo 설정
└── ...                     # 기타 설정 파일들
```

## 주요 명령어

### 설치 및 초기 설정

```bash
# 모든 의존성 설치
pnpm install

# 특정 애플리케이션에 패키지 추가
pnpm add <package-name> --filter=@project/app-name
```

### 개발 및 빌드

```bash
# 모든 프로젝트 개발 서버 실행
pnpm dev

# 특정 애플리케이션만 실행
pnpm dev --filter=@project/app-name

# 모든 프로젝트 빌드
pnpm build

# 특정 프로젝트만 빌드
pnpm build --filter=@project/app-name
```

### 코드 품질

```bash
# 모든 프로젝트 린트 검사
pnpm lint

# 모든 프로젝트 포맷팅
pnpm format

# 타입 검사
pnpm typecheck
```

## 시작하기

### 🚀 빠른 시작
- **5분 만에 시작**: [quick-start.md](./quick-start.md) - 검증된 설정으로 빠르게 시작
- **문제 해결**: [troubleshooting.md](./troubleshooting.md) - 자주 발생하는 문제와 해결방법

### 📚 상세 가이드
1. **워크스페이스 설정**: [workspace.md](./workspace.md)를 참고하여 기본 설정을 완료하세요
2. **설정 관리**: [config.md](./config.md)를 참고하여 공유 설정을 구성하세요
3. **앱 개발**: [apps.md](./apps.md)를 참고하여 애플리케이션을 구성하세요
4. **패키지 개발**: [packages.md](./packages.md)를 참고하여 공유 패키지를 만드세요

### 🔧 특별 주제
- **React 19 설정**: [react19-setup.md](./react19-setup.md) - React 19 사용 시 주의사항과 설정

## 추가 자료

- [Turborepo 공식 문서](https://turbo.build/repo/docs)
- [pnpm 워크스페이스 가이드](https://pnpm.io/workspaces)
- [모노레포 모범 사례](https://monorepo.tools/)
