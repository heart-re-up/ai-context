# Husky - Git Hooks 자동화

Husky는 Git hooks를 쉽게 관리할 수 있게 해주는 도구입니다. 커밋, 푸시 등의 Git 작업 시 자동으로 스크립트를 실행합니다.

## 설치

```bash
pnpm add -D husky
```

## 초기 설정

```bash
# husky 초기화
pnpm exec husky init

# .husky 디렉토리가 생성되고 package.json에 prepare 스크립트 추가됨
```

## Hook 종류 및 용도

### pre-commit
커밋 전에 실행됩니다. 주로 코드 품질 검사에 사용합니다.

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# lint-staged 실행
pnpm lint-staged

# 테스트 실행 (선택사항)
# pnpm test:unit
```

### commit-msg
커밋 메시지 작성 후 실행됩니다. 커밋 메시지 규칙 검증에 사용합니다.

```bash
# .husky/commit-msg
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# commitlint 실행
npx --no -- commitlint --edit $1
```

### pre-push
푸시 전에 실행됩니다. 전체 테스트나 빌드 검증에 사용합니다.

```bash
# .husky/pre-push
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# 전체 테스트 실행
pnpm test

# 타입 체크
pnpm typecheck
```

## 고급 설정

### 조건부 실행

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# CI 환경에서는 실행하지 않음
[ -n "$CI" ] && exit 0

# 특정 브랜치에서만 실행
branch=$(git rev-parse --abbrev-ref HEAD)
if [ "$branch" = "main" ]; then
  pnpm test:e2e
fi

pnpm lint-staged
```

### Hook 비활성화

```bash
# 일시적으로 hooks 비활성화
HUSKY=0 git commit -m "skip hooks"

# 또는
git commit -m "skip hooks" --no-verify
```

## 모노레포에서의 Husky

### 루트 레벨 설정

```json
// package.json (root)
{
  "scripts": {
    "prepare": "husky install",
    "lint:all": "pnpm -r lint",
    "test:all": "pnpm -r test"
  }
}
```

### 워크스페이스별 처리

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

# 변경된 워크스페이스만 검사
pnpm exec lint-staged --relative
```

## 문제 해결

### Hook이 실행되지 않을 때

```bash
# husky 재설치
rm -rf .husky
pnpm exec husky init
```

### 권한 문제

```bash
# 실행 권한 부여
chmod +x .husky/*
```

### Windows에서의 이슈

```bash
# Git Bash 사용 권장
# 또는 .husky 파일에 CRLF -> LF 변환
```

## Best Practices

1. **빠른 실행**: pre-commit은 빠르게 실행되어야 함
2. **명확한 에러**: 실패 시 명확한 메시지 출력
3. **선택적 실행**: CI 환경에서는 비활성화
4. **팀 공유**: .husky 디렉토리를 Git에 포함

## 다른 도구와의 연동

- **lint-staged**: 변경된 파일만 검사
- **commitlint**: 커밋 메시지 검증
- **prettier**: 코드 포맷팅
- **eslint**: 코드 품질 검사
