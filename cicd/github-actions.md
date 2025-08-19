# GitHub Actions 워크플로우

GitHub Actions를 활용한 CI/CD 파이프라인 구성 가이드입니다.

## 기본 워크플로우

### 코드 품질 검증

```yaml
# .github/workflows/quality.yml
name: Code Quality

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

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
```

### 빌드 및 배포

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build:
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
        
      - name: Build
        run: pnpm build
        
      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-files
          path: dist/

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-files
          path: dist/
          
      - name: Deploy to production
        run: |
          # 배포 스크립트 실행
          ./scripts/deploy.sh
```

## 모노레포 워크플로우

### 변경된 패키지만 빌드

```yaml
# .github/workflows/monorepo.yml
name: Monorepo CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

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

  test-packages:
    needs: changes
    if: ${{ needs.changes.outputs.packages == 'true' }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm test --filter="./packages/*"

  build-apps:
    needs: changes
    if: ${{ needs.changes.outputs.apps == 'true' }}
    runs-on: ubuntu-latest
    strategy:
      matrix:
        app: [web-app, admin-dashboard]
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm build --filter=@project/${{ matrix.app }}
```

## Docker 통합

### Docker 이미지 빌드

```yaml
# .github/workflows/docker.yml
name: Docker Build

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  docker:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Docker Buildx
        uses: docker/setup-buildx-action@v3
        
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
          
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            myapp/app:latest
            myapp/app:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

## 환경별 배포

### 환경 구분

```yaml
# .github/workflows/deploy-env.yml
name: Environment Deploy

on:
  push:
    branches:
      - main        # production
      - develop     # staging
      - feature/*   # development

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
          elif [[ ${{ github.ref }} == 'refs/heads/develop' ]]; then
            echo "environment=staging" >> $GITHUB_OUTPUT
          else
            echo "environment=development" >> $GITHUB_OUTPUT
          fi
          
      - name: Deploy
        run: |
          echo "Deploying to ${{ steps.env.outputs.environment }}"
          ./scripts/deploy-${{ steps.env.outputs.environment }}.sh
```

## 보안 설정

### Secrets 관리

```yaml
# 환경변수와 시크릿 사용
env:
  NODE_ENV: production
  API_URL: ${{ secrets.API_URL }}
  DATABASE_URL: ${{ secrets.DATABASE_URL }}
  JWT_SECRET: ${{ secrets.JWT_SECRET }}
```

### 의존성 보안 검사

```yaml
# .github/workflows/security.yml
name: Security Check

on:
  schedule:
    - cron: '0 2 * * 1'  # 매주 월요일 오전 2시
  push:
    branches: [main]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run security audit
        run: pnpm audit
        
      - name: Check for vulnerabilities
        uses: actions/dependency-review-action@v3
        if: github.event_name == 'pull_request'
```

## 성능 최적화

### 캐시 활용

```yaml
# 의존성 캐시
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: 18
    cache: "pnpm"

# ESLint 캐시
- name: Cache ESLint
  uses: actions/cache@v3
  with:
    path: .eslintcache
    key: eslint-${{ hashFiles('**/eslint.config.mjs') }}

# 빌드 캐시
- name: Cache build
  uses: actions/cache@v3
  with:
    path: |
      .turbo
      **/dist
    key: build-${{ hashFiles('**/package.json') }}-${{ hashFiles('**/*.ts', '**/*.tsx') }}
```

### 병렬 작업

```yaml
jobs:
  test:
    strategy:
      matrix:
        node-version: [18, 20]
        os: [ubuntu-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm test
```

## 실용적인 팁

### 조건부 실행

```yaml
# 특정 파일 변경시만 실행
- name: Build frontend
  if: contains(github.event.head_commit.modified, 'apps/web/')
  run: pnpm build --filter=@project/web

# PR에서만 실행
- name: Preview deployment
  if: github.event_name == 'pull_request'
  run: ./scripts/deploy-preview.sh
```

### 에러 처리

```yaml
- name: Test with retry
  uses: nick-invision/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    command: pnpm test

- name: Notify on failure
  if: failure()
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

## 템플릿

### 기본 CI 템플릿

```yaml
# templates/basic-ci.yml
name: CI

on: [push, pull_request]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm test
      - run: pnpm build
```

### 모노레포 템플릿

```yaml
# templates/monorepo-ci.yml
name: Monorepo CI

on: [push, pull_request]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm test
      - run: pnpm build
```

## 문제 해결

### 자주 발생하는 문제

1. **의존성 설치 실패**
   ```yaml
   - run: pnpm install --frozen-lockfile --prefer-offline
   ```

2. **메모리 부족**
   ```yaml
   - run: NODE_OPTIONS="--max-old-space-size=4096" pnpm build
   ```

3. **권한 문제**
   ```yaml
   - run: chmod +x ./scripts/deploy.sh
   ```

## 모범 사례

1. **워크플로우 분리**: CI와 CD를 별도 파일로 관리
2. **환경별 설정**: 환경변수와 시크릿으로 환경 구분
3. **실패 시 알림**: Slack, 이메일 등으로 실패 알림 설정
4. **아티팩트 관리**: 빌드 결과물을 아티팩트로 저장
5. **보안 강화**: 시크릿 사용, 의존성 보안 검사