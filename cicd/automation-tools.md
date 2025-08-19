# 자동화 도구 연동

CI/CD 파이프라인에서 활용할 수 있는 자동화 도구들을 다룹니다.

## 의존성 관리 자동화

### Renovate 연동

```json
// renovate.json - CI/CD 친화적 설정
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:base"],
  "timezone": "Asia/Seoul",
  "schedule": ["before 9am on monday"],
  
  "packageRules": [
    {
      "matchUpdateTypes": ["patch"],
      "automerge": true,
      "automergeType": "branch"
    },
    {
      "matchUpdateTypes": ["minor"],
      "automerge": true,
      "automergeType": "pr",
      "requiredStatusChecks": ["ci/quality", "ci/test"]
    },
    {
      "matchUpdateTypes": ["major"],
      "automerge": false,
      "labels": ["major-update"],
      "reviewers": ["@tech-lead"]
    }
  ],
  
  "vulnerabilityAlerts": {
    "enabled": true,
    "automerge": true,
    "schedule": ["at any time"]
  }
}
```

### GitHub Actions에서 Renovate 트리거

```yaml
# .github/workflows/renovate.yml
name: Renovate

on:
  schedule:
    - cron: '0 2 * * 1'  # 매주 월요일 오전 2시
  workflow_dispatch:

jobs:
  renovate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        
      - name: Self-hosted Renovate
        uses: renovatebot/github-action@v39.2.4
        with:
          configurationFile: renovate.json
          token: ${{ secrets.RENOVATE_TOKEN }}
```

## 릴리스 자동화

### Changesets 연동

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
          
      - name: Install dependencies
        run: pnpm install --frozen-lockfile
        
      - name: Build packages
        run: pnpm build
        
      - name: Create Release PR or Publish
        uses: changesets/action@v1
        with:
          publish: pnpm changeset publish
          version: pnpm changeset version
          commit: "chore: release packages"
          title: "chore: release packages"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

### Semantic Release 연동

```yaml
# .github/workflows/semantic-release.yml
name: Semantic Release

on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          persist-credentials: false
          
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          
      - name: Install dependencies
        run: npm ci
        
      - name: Build
        run: npm run build
        
      - name: Semantic Release
        uses: cycjimmy/semantic-release-action@v3
        with:
          extra_plugins: |
            @semantic-release/changelog
            @semantic-release/git
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## 코드 품질 자동화

### Husky + lint-staged CI 연동

```yaml
# CI에서 Git hooks 무시
- name: Install dependencies
  run: |
    export HUSKY=0  # CI에서 husky 비활성화
    pnpm install --frozen-lockfile

# 또는 CI 전용 스크립트 사용
- name: Quality check
  run: pnpm quality:ci  # husky 없이 동일한 검사 실행
```

### 자동 코드 리뷰

```yaml
# .github/workflows/code-review.yml
name: Automated Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
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
        
      - name: Run quality checks
        run: |
          pnpm lint --format json --output-file eslint-report.json
          pnpm test --coverage --reporter=json --outputFile=test-report.json
          
      - name: Comment PR with results
        uses: actions/github-script@v6
        with:
          script: |
            const eslint = require('./eslint-report.json');
            const tests = require('./test-report.json');
            
            const errors = eslint.reduce((acc, file) => acc + file.errorCount, 0);
            const warnings = eslint.reduce((acc, file) => acc + file.warningCount, 0);
            
            const comment = `
            ## 🤖 Automated Code Review
            
            ### ESLint Results
            - Errors: ${errors}
            - Warnings: ${warnings}
            
            ### Test Results  
            - Tests: ${tests.numTotalTests}
            - Passed: ${tests.numPassedTests}
            - Coverage: ${tests.coverageMap ? '✅' : '❌'}
            
            ${errors > 0 ? '❌ Please fix ESLint errors before merging' : '✅ Code quality looks good!'}
            `;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });
```

## 배포 자동화

### 환경별 자동 배포

```yaml
# .github/workflows/auto-deploy.yml
name: Auto Deploy

on:
  push:
    branches:
      - main      # production
      - develop   # staging

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Determine environment
        id: env
        run: |
          if [[ ${{ github.ref }} == 'refs/heads/main' ]]; then
            echo "environment=production" >> $GITHUB_OUTPUT
            echo "url=https://app.example.com" >> $GITHUB_OUTPUT
          else
            echo "environment=staging" >> $GITHUB_OUTPUT
            echo "url=https://staging.example.com" >> $GITHUB_OUTPUT
          fi
          
      - name: Deploy to ${{ steps.env.outputs.environment }}
        run: |
          ./scripts/deploy.sh ${{ steps.env.outputs.environment }}
          
      - name: Health check
        run: |
          sleep 30  # 배포 대기
          curl -f ${{ steps.env.outputs.url }}/health
          
      - name: Notify deployment
        run: |
          curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
            -H 'Content-type: application/json' \
            -d "{\"text\":\"🚀 Deployed to ${{ steps.env.outputs.environment }}: ${{ steps.env.outputs.url }}\"}"
```

## 모니터링 연동

### 배포 후 모니터링

```yaml
# 배포 후 자동 모니터링
- name: Setup monitoring
  run: |
    # 배포 마커 추가 (APM 도구용)
    curl -X POST "${{ secrets.APM_WEBHOOK }}" \
      -H "Content-Type: application/json" \
      -d "{
        \"deployment\": {
          \"revision\": \"${{ github.sha }}\",
          \"environment\": \"production\",
          \"timestamp\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"
        }
      }"

# 배포 후 헬스체크
- name: Post-deployment health check
  run: |
    for i in {1..10}; do
      if curl -f https://app.example.com/health; then
        echo "✅ Health check passed"
        break
      fi
      echo "⏳ Waiting for service... ($i/10)"
      sleep 30
    done
```

## 알림 자동화

### 다양한 알림 채널

```yaml
# 성공/실패에 따른 다른 알림
- name: Notify success
  if: success()
  uses: 8398a7/action-slack@v3
  with:
    status: success
    channel: '#deployments'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
    message: |
      ✅ Deployment successful
      Environment: ${{ steps.env.outputs.environment }}
      Commit: ${{ github.sha }}

- name: Notify failure
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    channel: '#alerts'
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
    message: |
      🚨 Deployment failed
      Please check the logs: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}
```

## 롤백 자동화

### 자동 롤백 트리거

```yaml
# 배포 실패시 자동 롤백
- name: Auto rollback on failure
  if: failure()
  run: |
    echo "🔄 Starting automatic rollback..."
    
    # 이전 성공한 배포 찾기
    LAST_SUCCESS=$(git log --format="%H" --grep="deploy: success" -1)
    
    # 롤백 실행
    ./scripts/rollback.sh $LAST_SUCCESS
    
    # 롤백 알림
    curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
      -d "{\"text\":\"🔄 Auto rollback completed to commit: $LAST_SUCCESS\"}"
```

## 실용적인 스크립트

### 통합 자동화 스크립트

```bash
#!/bin/bash
# scripts/ci-pipeline.sh

set -e

echo "🚀 Starting CI pipeline..."

# 1. 코드 품질 검사
echo "📋 Quality checks..."
pnpm lint
pnpm format
pnpm type-check

# 2. 테스트 실행
echo "🧪 Running tests..."
pnpm test --coverage

# 3. 빌드
echo "🔨 Building..."
pnpm build

# 4. 보안 검사
echo "🔒 Security audit..."
pnpm audit --audit-level high

echo "✅ CI pipeline completed successfully!"
```

### 배포 스크립트

```bash
#!/bin/bash
# scripts/deploy.sh

ENVIRONMENT=${1:-staging}
VERSION=${2:-latest}

echo "🚀 Deploying to $ENVIRONMENT..."

# 환경별 설정 로드
source .env.$ENVIRONMENT

# Docker 이미지 빌드 및 배포
docker build -t myapp:$VERSION .
docker tag myapp:$VERSION $REGISTRY/myapp:$VERSION
docker push $REGISTRY/myapp:$VERSION

# 컨테이너 업데이트
docker service update --image $REGISTRY/myapp:$VERSION myapp-service

echo "✅ Deployment to $ENVIRONMENT completed!"
```

## 모범 사례

1. **점진적 자동화**: 수동 → 반자동 → 완전 자동화
2. **실패 시 알림**: 즉시 팀에 알림 전송
3. **롤백 준비**: 언제든 이전 버전으로 복구 가능
4. **모니터링 연동**: 배포 후 자동 상태 확인
5. **문서화**: 모든 자동화 과정을 문서로 기록