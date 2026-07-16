# Node.js 버전 관리 가이드

Node.js와 패키지 매니저 버전을 팀원 간에 동기화하는 방법에 대한 가이드입니다.

## 버전 관리의 중요성

- **일관성**: 모든 팀원이 동일한 Node.js 버전 사용
- **안정성**: 프로덕션과 개발 환경의 일치
- **호환성**: 패키지 의존성과 Node.js 버전 간 호환성 보장

## Node.js 버전 고정 방법

### .nvmrc (NVM 사용자용)

```
18.19.0
```

**사용법:**

```bash
# 프로젝트 디렉토리에서
nvm use

# 자동으로 해당 버전 사용
nvm install
```

### .node-version (nodenv/asdf 사용자용)

```
18.19.0
```

**사용법:**

```bash
# nodenv 사용자
nodenv install 18.19.0
nodenv local 18.19.0

# asdf 사용자
asdf install nodejs 18.19.0
asdf local nodejs 18.19.0
```

### package.json engines 필드

```json
{
  "engines": {
    "node": ">=18.19.0 <19",
    "pnpm": ">=8.0.0"
  }
}
```

**효과:**

- npm/pnpm 설치 시 버전 체크
- CI/CD에서 버전 검증
- 명시적인 요구사항 문서화

## Volta 설정 (권장)

### 설치

```bash
# macOS/Linux
curl https://get.volta.sh | bash

# Windows
curl https://get.volta.sh | bash
```

### 프로젝트 설정

```json
// package.json
{
  "volta": {
    "node": "18.19.0",
    "pnpm": "8.15.0"
  }
}
```

### Volta 장점

1. **자동 전환**: 프로젝트 디렉토리 진입 시 자동으로 버전 전환
2. **빠른 속도**: 바이너리 캐싱으로 빠른 전환
3. **패키지 매니저 포함**: Node.js뿐만 아니라 npm, pnpm도 관리
4. **크로스 플랫폼**: Windows, macOS, Linux 지원

### Volta 명령어

```bash
# Node.js 버전 고정
volta pin node@18.19.0

# pnpm 버전 고정
volta pin pnpm@8.15.0

# 현재 설정 확인
volta list

# 프로젝트 설정 확인
volta which node
volta which pnpm
```

## 팀 워크플로우

### 1. 프로젝트 초기 설정

```bash
# 1. Volta로 Node.js 버전 고정
volta pin node@18.19.0
volta pin pnpm@8.15.0

# 2. .nvmrc 파일 생성 (Volta 미사용자를 위해)
echo "18.19.0" > .nvmrc

# 3. .node-version 파일 생성 (nodenv/asdf 사용자를 위해)
echo "18.19.0" > .node-version
```

### 2. 새로운 팀원 온보딩

```bash
# Option 1: Volta 사용 (권장)
volta install node pnpm

# Option 2: NVM 사용
nvm install
nvm use

# Option 3: nodenv 사용
nodenv install $(cat .node-version)
nodenv local $(cat .node-version)

# 의존성 설치
pnpm install
```

### 3. CI/CD 설정

```yaml
# GitHub Actions 예시
name: CI
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Option 1: Volta 사용
      - uses: volta-cli/action@v4

      # Option 2: Node.js 버전 직접 지정
      - uses: actions/setup-node@v4
        with:
          node-version-file: ".nvmrc"
          cache: "pnpm"

      - name: Install dependencies
        run: pnpm install --frozen-lockfile
```

## 버전 업그레이드 전략

### 1. LTS 버전 정책

```json
{
  "engines": {
    "node": ">=18.19.0 <20"
  }
}
```

- 안정성을 위해 LTS(Long Term Support) 버전 사용
- 메이저 버전 업그레이드는 신중히 진행

### 2. 점진적 업그레이드

```bash
# 1. 로컬에서 새 버전 테스트
volta install node@20.10.0
volta pin node@20.10.0

# 2. 테스트 실행
pnpm test
pnpm build

# 3. 문제없으면 팀에 공유
git add package.json .nvmrc .node-version
git commit -m "chore: upgrade Node.js to 20.10.0"
```

### 3. 호환성 체크리스트

- [ ] 모든 패키지가 새 Node.js 버전 지원
- [ ] 빌드 도구 호환성 확인
- [ ] 테스트 통과 확인
- [ ] 프로덕션 환경 테스트

## 문제 해결

### 버전 불일치 이슈

```bash
# 현재 버전 확인
node --version
pnpm --version

# Volta 설정 확인
volta list

# 캐시 클리어
volta install node@18.19.0 --force
```

### 패키지 설치 오류

```bash
# Node.js 버전 확인 후 재설치
rm -rf node_modules package-lock.json
pnpm install
```

### CI/CD 빌드 실패

1. `.nvmrc` 파일 존재 확인
2. `package.json` engines 필드 확인
3. CI 환경의 Node.js 버전 로그 확인

## 모범 사례

1. **일관성**: 모든 환경에서 동일한 버전 사용
2. **문서화**: README에 필요한 Node.js 버전 명시
3. **자동화**: Volta 등 도구로 수동 작업 최소화
4. **검증**: engines 필드로 버전 요구사항 강제
5. **백업**: 여러 버전 관리 파일 동시 제공 (.nvmrc, .node-version)

## 관련 문서

- [monorepo/workspace.md](../monorepo/workspace.md) - 워크스페이스 패키지 매니저 설정
- [scripts.md](./scripts.md) - Node.js 프로젝트 스크립트 관리
