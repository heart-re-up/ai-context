# 자동화 설정 가이드

프로젝트의 개발 워크플로우를 자동화하기 위한 도구들과 설정 방법을 안내합니다.

## 범용 자동화 도구

### 의존성 관리

- **[Renovate](./renovate.md)**: 크로스 플랫폼 의존성 자동 업데이트
- **[Changesets](./changesets.md)**: 모노레포 버전 관리 및 릴리스

### 코드 품질

- **[Husky](./husky.md)**: Git Hooks 관리로 커밋/푸시 시 자동 검증
- **[Commitlint](./commitlint.md)**: 커밋 메시지 규칙 검증
- **[Lint-staged](./lint-staged.md)**: Staged 파일만 선택적 검증

## CI/CD 통합

자동화 도구들을 CI/CD 파이프라인과 연동하는 방법은 별도 가이드를 참고하세요:

👉 **[../cicd/README.md](../cicd/README.md)** - CI/CD 파이프라인 가이드

### 주요 연동 도구

- **[GitHub Actions](../cicd/github-actions.md)**: GitHub 호스팅 프로젝트
- **[Azure Pipelines](../cicd/azure-pipelines.md)**: Azure DevOps 환경  
- **[자동화 도구 연동](../cicd/automation-tools.md)**: Renovate, Changesets 등

## 도구 선택 가이드

### 프로젝트 규모별 추천

#### 소규모 프로젝트

- **필수**: Husky + Prettier
- **권장**: Commitlint

#### 중규모 프로젝트

- **필수**: Husky + Commitlint + Lint-staged
- **권장**: Renovate

#### 대규모/모노레포 프로젝트

- **필수**: 모든 도구
- **추가**: Changesets (버전 관리)

### 단계별 도입 전략

1. **1단계 - 기본 품질 관리**

   ```bash
   pnpm add -D husky lint-staged prettier
   ```

   - 기본적인 코드 포맷팅과 린팅

2. **2단계 - 커밋 규칙**

   ```bash
   pnpm add -D @commitlint/{cli,config-conventional}
   ```

   - 일관된 커밋 메시지 유지

3. **3단계 - 의존성 자동화**

   - Renovate 또는 Dependabot 설정
   - 보안 업데이트 자동화

4. **4단계 - 릴리스 자동화**
   ```bash
   pnpm add -D @changesets/cli
   ```
   - 버전 관리와 릴리스 노트 자동화

## 빠른 시작

### 모든 도구 한 번에 설정

```bash
# 의존성 설치
pnpm add -D husky lint-staged @commitlint/{cli,config-conventional}

# Husky 초기화
pnpm exec husky init

# Git hooks 설정
echo 'pnpm exec lint-staged' > .husky/pre-commit
echo 'npx --no -- commitlint --edit $1' > .husky/commit-msg

# 설정 파일 생성
touch .lintstagedrc.js commitlint.config.js
```

### 기본 설정 파일

**.lintstagedrc.js**

```javascript
module.exports = {
  "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"],
  "*.{json,md,yml}": ["prettier --write"],
};
```

**commitlint.config.js**

```javascript
module.exports = {
  extends: ["@commitlint/config-conventional"],
};
```

## 모범 사례

1. **점진적 도입**: 한 번에 모든 도구를 도입하지 말고 단계적으로 적용
2. **팀 교육**: 새로운 도구 도입 시 팀원들에게 충분한 설명과 교육 제공
3. **문서화**: 프로젝트별 커스터마이징 내용은 반드시 문서화
4. **정기 검토**: 도구 설정을 정기적으로 검토하고 개선
5. **성능 모니터링**: 자동화 도구가 개발 속도를 저해하지 않는지 확인

## 트러블슈팅

### 공통 이슈

1. **Git hooks가 작동하지 않음**

   ```bash
   chmod +x .husky/*
   ```

2. **Windows 환경 이슈**

   - Git Bash 사용 권장
   - CRLF/LF 설정 확인

3. **CI 환경에서 불필요한 실행**
   - 환경 변수로 조건부 실행 설정

## 참고 자료

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)
- [Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
