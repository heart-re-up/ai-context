# 품질 관리 파이프라인

CI/CD에서의 코드 품질 검증 및 자동화 전략을 다룹니다.

## 품질 검증 단계

### 기본 품질 체크

```yaml
# GitHub Actions 예시
name: Quality Check

on: [push, pull_request]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
          
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
        
      - name: Type check
        run: pnpm type-check
        
      - name: Lint
        run: pnpm lint
        
      - name: Format check
        run: pnpm format
        
      - name: Test
        run: pnpm test --coverage
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info
```

### Azure Pipelines 예시

```yaml
# azure-pipelines-quality.yml
trigger: [main, develop]
pr: [main]

pool:
  vmImage: 'ubuntu-latest'

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '18.x'

  - script: |
      npm install -g pnpm
      pnpm install --frozen-lockfile
    displayName: 'Install dependencies'

  - script: pnpm type-check
    displayName: 'TypeScript check'

  - script: pnpm lint
    displayName: 'ESLint check'

  - script: pnpm format
    displayName: 'Prettier check'

  - script: pnpm test --coverage --reporter=junit
    displayName: 'Run tests'

  - task: PublishTestResults@2
    condition: succeededOrFailed()
    inputs:
      testResultsFiles: 'junit.xml'
      testRunTitle: 'Unit Tests'

  - task: PublishCodeCoverageResults@1
    inputs:
      codeCoverageTool: 'Cobertura'
      summaryFileLocation: 'coverage/cobertura-coverage.xml'
```

## 병렬 품질 검증

### 작업 분리

```yaml
# GitHub Actions - 병렬 실행
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint

  format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm format

  type-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm type-check

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm test --coverage

  build:
    needs: [lint, format, type-check, test]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm build
```

## 모노레포 품질 관리

### 변경된 패키지만 검증

```yaml
# 효율적인 모노레포 품질 검증
name: Monorepo Quality

on: [push, pull_request]

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      packages: ${{ steps.changes.outputs.packages }}
      apps: ${{ steps.changes.outputs.apps }}
    steps:
      - uses: actions/checkout@v4
      - uses: dorny/paths-filter@v2
        id: changes
        with:
          filters: |
            packages:
              - 'packages/**'
            apps:
              - 'apps/**'

  quality-packages:
    needs: changes
    if: ${{ needs.changes.outputs.packages == 'true' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint --filter="./packages/*"
      - run: pnpm test --filter="./packages/*"

  quality-apps:
    needs: changes
    if: ${{ needs.changes.outputs.apps == 'true' }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        app: [web-app, admin-dashboard]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint --filter=@project/${{ matrix.app }}
      - run: pnpm test --filter=@project/${{ matrix.app }}
```

## 보안 검증

### 의존성 보안 검사

```yaml
# 보안 취약점 검사
- name: Security audit
  run: pnpm audit

- name: Dependency review
  uses: actions/dependency-review-action@v3
  if: github.event_name == 'pull_request'

- name: CodeQL analysis
  uses: github/codeql-action/init@v2
  with:
    languages: typescript, javascript
```

### 라이선스 검사

```yaml
# 라이선스 호환성 검사
- name: License check
  run: |
    pnpm install -g license-checker
    license-checker --onlyAllow 'MIT;Apache-2.0;BSD-3-Clause' --production
```

## 성능 최적화

### 캐시 전략

```yaml
# 효율적인 캐시 사용
- name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: |
      ~/.pnpm-store
      .turbo
    key: ${{ runner.os }}-pnpm-${{ hashFiles('**/pnpm-lock.yaml') }}
    restore-keys: |
      ${{ runner.os }}-pnpm-

- name: Cache ESLint
  uses: actions/cache@v3
  with:
    path: .eslintcache
    key: ${{ runner.os }}-eslint-${{ hashFiles('**/eslint.config.mjs') }}
```

### 조건부 실행

```yaml
# 필요한 경우만 실행
- name: Run tests
  if: contains(github.event.head_commit.message, '[test]') || github.event_name == 'pull_request'
  run: pnpm test

- name: Type check
  if: contains(github.event.head_commit.modified, '*.ts') || contains(github.event.head_commit.modified, '*.tsx')
  run: pnpm type-check
```

## 품질 게이트

### 필수 통과 조건

```yaml
# 품질 기준 설정
- name: Quality gate
  run: |
    # 테스트 커버리지 80% 이상
    COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')
    if (( $(echo "$COVERAGE < 80" | bc -l) )); then
      echo " Coverage is below 80%: $COVERAGE%"
      exit 1
    fi
    
    # ESLint 에러 0개
    pnpm lint --format json --output-file eslint-report.json
    ERRORS=$(cat eslint-report.json | jq '[.[].errorCount] | add')
    if [ "$ERRORS" -gt 0 ]; then
      echo " ESLint errors found: $ERRORS"
      exit 1
    fi
    
    echo " Quality gate passed"
```

### 브랜치 보호 규칙

```yaml
# GitHub에서 브랜치 보호 설정
# Settings > Branches > Add rule
# - Require status checks to pass before merging
# - Require branches to be up to date before merging
# - Include administrators
```

## 리포트 생성

### 품질 리포트

```yaml
# 품질 리포트 생성 및 저장
- name: Generate quality report
  run: |
    mkdir -p reports
    
    # ESLint 리포트
    pnpm lint --format json --output-file reports/eslint.json
    
    # 테스트 커버리지
    cp coverage/coverage-summary.json reports/
    
    # 번들 크기 분석 (선택사항)
    pnpm build:analyze > reports/bundle-analysis.txt

- name: Upload reports
  uses: actions/upload-artifact@v4
  with:
    name: quality-reports
    path: reports/
```

### 품질 트렌드 추적

```yaml
# 품질 메트릭 추적
- name: Track quality metrics
  run: |
    echo "timestamp,coverage,eslint_errors,bundle_size" >> quality-metrics.csv
    echo "$(date +%s),$(cat coverage/coverage-summary.json | jq '.total.lines.pct'),$(cat reports/eslint.json | jq '[.[].errorCount] | add'),$(stat -c%s dist/main.js)" >> quality-metrics.csv

- name: Commit metrics
  run: |
    git config --local user.email "action@github.com"
    git config --local user.name "GitHub Action"
    git add quality-metrics.csv
    git commit -m "chore: update quality metrics [skip ci]" || exit 0
    git push
```

## 자동 수정

### 코드 포맷팅 자동 수정

```yaml
# PR에서 자동 포맷팅
- name: Auto-fix formatting
  run: |
    pnpm format:fix
    pnpm lint:fix

- name: Commit fixes
  run: |
    git config --local user.email "action@github.com"
    git config --local user.name "GitHub Action"
    git add .
    git diff --staged --quiet || git commit -m "style: auto-fix formatting and linting"
    git push
```

## 알림 및 피드백

### 실패 시 알림

```yaml
# Slack 알림
- name: Notify on failure
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    channel: '#dev-alerts'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
    message: |
       Quality check failed
      Repository: ${{ github.repository }}
      Branch: ${{ github.ref }}
      Commit: ${{ github.sha }}
```

### PR 코멘트

```yaml
# PR에 품질 리포트 코멘트
- name: Comment PR
  if: github.event_name == 'pull_request'
  uses: actions/github-script@v6
  with:
    script: |
      const coverage = require('./coverage/coverage-summary.json');
      const comment = `
      ##  Quality Report
      
      - **Test Coverage**: ${coverage.total.lines.pct}%
      - **ESLint Errors**: 0
      - **Build Status**:  Passed
      `;
      
      github.rest.issues.createComment({
        issue_number: context.issue.number,
        owner: context.repo.owner,
        repo: context.repo.repo,
        body: comment
      });
```

## 실용적인 스크립트

### package.json 스크립트

```json
{
  "scripts": {
    "quality": "pnpm lint && pnpm format && pnpm type-check && pnpm test",
    "quality:fix": "pnpm lint:fix && pnpm format:fix",
    "quality:ci": "pnpm lint --format json --output-file eslint-report.json && pnpm format && pnpm type-check && pnpm test --coverage --reporter=junit",
    "pre-commit": "lint-staged",
    "pre-push": "pnpm quality"
  }
}
```

### Git Hooks 연동

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

echo " Running quality checks..."

# 변경된 파일만 검사
pnpm lint-staged

# 타입 체크
pnpm type-check

echo " Quality checks passed!"
```

## 모범 사례

### 단계별 품질 관리

1. **개발 중**: 에디터 실시간 검사
2. **커밋 전**: Git hooks로 변경 파일만 검사
3. **PR**: 전체 코드베이스 품질 검증
4. **배포 전**: 최종 품질 게이트 통과

### 효율적인 검사 순서

```yaml
# 빠른 검사부터 실행하여 조기 실패
steps:
  - name: Format check (가장 빠름)
    run: pnpm format
    
  - name: Lint check (빠름)
    run: pnpm lint
    
  - name: Type check (보통)
    run: pnpm type-check
    
  - name: Test (가장 오래 걸림)
    run: pnpm test --coverage
```

### 실패 시 빠른 피드백

```yaml
# 실패시 즉시 중단
- name: Quality checks
  run: |
    set -e  # 에러 발생시 즉시 중단
    pnpm lint
    pnpm format
    pnpm type-check
    pnpm test
```

## 품질 메트릭

### 추적할 지표

- **테스트 커버리지**: 80% 이상 유지
- **ESLint 에러**: 0개 유지
- **타입 에러**: 0개 유지
- **빌드 시간**: 성능 모니터링
- **번들 크기**: 크기 증가 추적

### 품질 대시보드

```yaml
# 품질 메트릭 수집
- name: Collect metrics
  run: |
    echo "COVERAGE=$(cat coverage/coverage-summary.json | jq '.total.lines.pct')" >> $GITHUB_ENV
    echo "BUNDLE_SIZE=$(stat -c%s dist/main.js)" >> $GITHUB_ENV
    echo "BUILD_TIME=${{ steps.build.outputs.time }}" >> $GITHUB_ENV

- name: Update dashboard
  run: |
    curl -X POST "${{ secrets.DASHBOARD_WEBHOOK }}" \
      -H "Content-Type: application/json" \
      -d "{
        \"coverage\": $COVERAGE,
        \"bundle_size\": $BUNDLE_SIZE,
        \"build_time\": $BUILD_TIME,
        \"commit\": \"${{ github.sha }}\"
      }"
```

## 문제 해결

### 일반적인 문제

1. **메모리 부족**
   ```yaml
   - name: Build with more memory
     run: NODE_OPTIONS="--max-old-space-size=4096" pnpm build
   ```

2. **타임아웃**
   ```yaml
   - name: Test with timeout
     timeout-minutes: 10
     run: pnpm test
   ```

3. **캐시 문제**
   ```yaml
   - name: Clear cache
     run: |
       rm -rf .eslintcache
       rm -rf node_modules/.cache
   ```

## 자동화 팁

### 조건부 품질 검사

```yaml
# 특정 파일 변경시만 실행
- name: Check if frontend changed
  uses: dorny/paths-filter@v2
  id: changes
  with:
    filters: |
      frontend:
        - 'apps/web/**'
        - 'packages/ui/**'

- name: Frontend quality check
  if: steps.changes.outputs.frontend == 'true'
  run: pnpm lint --filter="./apps/web" --filter="./packages/ui"
```

### 점진적 품질 향상

```yaml
# 새 코드에만 엄격한 기준 적용
- name: Strict check for new code
  run: |
    # 변경된 파일만 엄격하게 검사
    git diff --name-only HEAD~1 HEAD | grep -E '\.(ts|tsx)$' | xargs pnpm eslint --max-warnings 0
```