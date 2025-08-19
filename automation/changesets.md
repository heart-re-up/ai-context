# Changesets - 모노레포 버전 관리 및 릴리스

Changesets는 모노레포에서 버전 관리와 릴리스를 자동화하는 도구입니다. 각 변경사항을 추적하고 자동으로 버전을 올리며 변경 로그를 생성합니다.

## 특징

- **모노레포 최적화**: 패키지 간 의존성 자동 처리
- **시맨틱 버저닝**: major, minor, patch 자동 관리
- **변경 로그 자동 생성**: 커밋 메시지 기반 CHANGELOG.md 생성
- **유연한 릴리스**: 수동/자동 릴리스 지원

## 설치

```bash
pnpm add -D @changesets/cli
```

## 초기 설정

```bash
# changesets 초기화
pnpm changeset init
```

이 명령은 `.changeset` 디렉토리와 `config.json` 파일을 생성합니다.

## 기본 설정

### .changeset/config.json

```json
{
  "$schema": "https://unpkg.com/@changesets/config@2.3.1/schema.json",
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "fixed": [],
  "linked": [],
  "access": "public",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": []
}
```

## 워크플로우

### 1. 변경사항 생성

```bash
# 대화형 모드로 changeset 생성
pnpm changeset

# 또는 직접 생성
pnpm changeset add
```

### 2. 생성된 changeset 파일

```markdown
---
"@project/ui": patch
"@project/utils": minor
---

Fix button hover state and add new date utility function
```

### 3. 버전 업데이트

```bash
# 버전 업데이트 (package.json 수정)
pnpm changeset version
```

### 4. 릴리스

```bash
# npm에 배포
pnpm changeset publish
```

## 고급 설정

### 패키지 그룹화 (Fixed versioning)

```json
{
  "fixed": [["@project/ui", "@project/utils", "@project/core"]]
}
```

모든 패키지가 같은 버전을 유지합니다.

### 연결된 패키지 (Linked versioning)

```json
{
  "linked": [["@project/eslint-*", "@project/prettier-*"]]
}
```

major/minor 업데이트 시 함께 버전이 올라갑니다.

### 커스텀 changelog

```json
{
  "changelog": ["@changesets/changelog-github", { "repo": "org/repo" }]
}
```

### 접근 권한 설정

```json
{
  "access": "public", // 또는 "restricted"

  // 패키지별 설정
  "access": {
    "@company/internal-*": "restricted",
    "@company/public-*": "public"
  }
}
```

## 스크립트 설정

### package.json (root)

```json
{
  "scripts": {
    "changeset": "changeset",
    "version": "changeset version && pnpm install --no-frozen-lockfile",
    "release": "pnpm build && changeset publish",
    "version:check": "changeset status --verbose",
    "version:snapshot": "changeset version --snapshot"
  }
}
```

## CI/CD 통합

### GitHub Actions

```yaml
name: Release

on:
  push:
    branches: [main]

concurrency: ${{ github.workflow }}-${{ github.ref }}

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"

      - run: pnpm install --frozen-lockfile

      - name: Create Release Pull Request or Publish
        uses: changesets/action@v1
        with:
          publish: pnpm release
          version: pnpm version
          commit: "chore: release"
          title: "chore: release"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## 사전 릴리스 (Pre-release)

### 알파/베타 버전

```bash
# pre 모드 진입
pnpm changeset pre enter alpha

# changeset 생성 및 버전 업데이트
pnpm changeset
pnpm changeset version

# 배포
pnpm changeset publish

# pre 모드 종료
pnpm changeset pre exit
```

### 스냅샷 릴리스

```bash
# 스냅샷 버전 생성
pnpm changeset version --snapshot

# 스냅샷 배포
pnpm changeset publish --tag canary
```

## 모노레포 설정

### 패키지 필터링

```json
{
  "ignore": ["@project/*-demo", "@project/playground"]
}
```

### 내부 의존성 업데이트

```json
{
  "updateInternalDependencies": "patch", // "patch" | "minor" | "major"

  // 또는 더 세밀한 제어
  "___experimentalUnsafeOptions_WILL_CHANGE_IN_PATCH": {
    "updateInternalDependents": "always" // "always" | "out-of-range"
  }
}
```

## 커스텀 워크플로우

### 자동 changeset 생성

```javascript
// scripts/create-changeset.js
const { writeFileSync } = require("fs");
const { join } = require("path");

const changeset = `---
"@project/ui": patch
---

Automated dependency updates
`;

const filename = `dep-update-${Date.now()}.md`;
writeFileSync(join(".changeset", filename), changeset);
```

### 버전 검증

```bash
# 현재 상태 확인
pnpm changeset status

# 상세 정보
pnpm changeset status --verbose
```

## 문제 해결

### changeset이 없을 때

```bash
# 빈 changeset 생성 (CI 스킵용)
pnpm changeset add --empty
```

### 버전 충돌

```bash
# 강제 버전 리셋
rm -rf .changeset
pnpm changeset init
```

### 배포 실패 복구

```bash
# 태그 정리
git tag -d v1.0.0
git push origin :refs/tags/v1.0.0

# 다시 배포
pnpm changeset publish
```

## Best Practices

1. **명확한 changeset**: 변경사항을 구체적으로 작성
2. **적절한 버전**: breaking changes는 major로
3. **그룹화 활용**: 관련 패키지는 함께 릴리스
4. **CI 자동화**: GitHub Actions로 자동 릴리스
5. **Pre-release 활용**: 중요 변경은 알파/베타 테스트

## 팀 워크플로우

### 1. Feature 브랜치

```bash
# feature 브랜치에서 작업
git checkout -b feature/new-component

# 작업 완료 후 changeset 생성
pnpm changeset
```

### 2. PR 리뷰

- changeset 파일 포함 확인
- 버전 타입 검토 (major/minor/patch)
- 설명 내용 확인

### 3. 자동 릴리스

- main 브랜치 머지 시 자동으로 Release PR 생성
- Release PR 머지 시 자동 배포

## 통합 예시

### 모노레포 전체 설정

```json
{
  "$schema": "https://unpkg.com/@changesets/config@2.3.1/schema.json",
  "changelog": ["@changesets/changelog-github", { "repo": "myorg/myrepo" }],
  "commit": false,
  "fixed": [],
  "linked": [["@project/eslint-*", "@project/prettier-*"]],
  "access": "public",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": ["@project/*-demo"],
  "___experimentalUnsafeOptions_WILL_CHANGE_IN_PATCH": {
    "onlyUpdatePeerDependentsWhenOutOfRange": true
  }
}
```
