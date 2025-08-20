# Next.js Docker 설정

Next.js 애플리케이션을 위한 Docker 설정 가이드입니다.

## 🤔 개발환경에서 Docker가 필요한 경우

**대부분의 경우 개발환경에서는 Docker가 불필요합니다:**

###  Docker 불필요한 일반적인 개발

```bash
# 로컬에서 직접 실행하는 것이 더 효율적
pnpm install
pnpm --filter=@project/web dev
```

###  Docker가 필요한 특수한 경우

1. **팀 환경 통일**: 모든 팀원이 동일한 개발환경이 필요할 때
2. **복잡한 마이크로서비스**: API, DB 등 여러 서비스 통합 개발
3. **SSR/API Routes**: Next.js의 서버 기능과 외부 서비스 연동
4. **프로덕션 환경 모방**: 정확한 프로덕션 환경 테스트가 필요할 때

## 개발환경 Docker (필요시에만)

```dockerfile
# Dockerfile.dev - 정말 필요한 경우에만 사용
FROM node:18-alpine AS base

# pnpm 설치
RUN corepack enable
RUN corepack prepare pnpm@8.15.0 --activate

WORKDIR /app

# 의존성 파일 복사
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/*/package.json apps/*/
COPY packages/*/package.json packages/*/

# 의존성 설치
RUN pnpm install --frozen-lockfile

# 소스 코드 복사
COPY . .

# Next.js 개발 서버 실행
EXPOSE 3000
CMD ["pnpm", "--filter=@project/web", "dev"]
```

##  권장: CI/CD 빌드 + Docker 배포

**가장 효율적인 방법:**

1. CI/CD에서 빌드 수행
2. 빌드 결과물만 Docker 이미지에 포함

CI/CD 파이프라인과 Docker 배포 전략에 대한 상세한 내용은 별도 가이드를 참고하세요:

 **[../cicd/docker-deployment.md](../cicd/docker-deployment.md)** - Docker 배포 전략 가이드

### 최적화된 프로덕션 Dockerfile

```dockerfile
# 이미 빌드된 결과물만 사용하는 가벼운 이미지
FROM node:18-alpine AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

# 비root 사용자 생성
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# CI/CD에서 빌드된 결과물 복사
COPY --chown=nextjs:nodejs apps/web/public ./public
COPY --chown=nextjs:nodejs apps/web/.next/standalone ./
COPY --chown=nextjs:nodejs apps/web/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

## 대안: 자체 빌드 포함 Dockerfile (권장하지 않음)

```dockerfile
# 만약 CI/CD 없이 Docker에서 직접 빌드해야 하는 경우
FROM node:18-alpine AS base

# pnpm 설치
RUN corepack enable
RUN corepack prepare pnpm@8.15.0 --activate

# Stage 1: 의존성 설치
FROM base AS deps
WORKDIR /app

COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/*/package.json apps/*/
COPY packages/*/package.json packages/*/

RUN pnpm install --frozen-lockfile

# Stage 2: 빌드
FROM base AS builder
WORKDIR /app

# 의존성 복사
COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/apps/*/node_modules ./apps/*/node_modules
COPY --from=deps /app/packages/*/node_modules ./packages/*/node_modules

# 소스 코드 복사
COPY . .

# Next.js 빌드
ENV NEXT_TELEMETRY_DISABLED 1
RUN pnpm --filter=@project/web build

# Stage 3: 실행
FROM node:18-alpine AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

# 비root 사용자 생성
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# 빌드 결과물 복사
COPY --from=builder --chown=nextjs:nodejs /app/apps/web/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/apps/web/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

## next.config.js 설정

Docker 배포를 위한 Next.js 설정:

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Docker standalone 빌드 활성화
  output: "standalone",

  // 이미지 최적화 설정
  images: {
    unoptimized: true, // 또는 외부 서비스 사용
  },

  // 환경변수 설정
  env: {
    CUSTOM_KEY: process.env.CUSTOM_KEY,
  },

  // API 경로 설정
  async rewrites() {
    return [
      {
        source: "/api/:path*",
        destination: `${process.env.API_URL}/api/:path*`,
      },
    ];
  },
};

module.exports = nextConfig;
```

## 개발용 docker-compose.yml (의존성 서비스만)

**권장 방법: 애플리케이션은 로컬에서, 의존성만 Docker로**

```yaml
# docker-compose.dev.yml - 의존성 서비스만
version: "3.8"

services:
  # 개발용 API (별도 컨테이너 필요시)
  api:
    image: myapi:dev # 또는 로컬에서 실행
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:postgres@postgres:5432/app_dev
    depends_on:
      - postgres

  # 개발용 데이터베이스
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app_dev
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

**사용법:**

```bash
# 의존성 서비스만 실행
docker-compose -f docker-compose.dev.yml up -d

# Next.js 앱은 로컬에서 실행
pnpm --filter=@project/web dev
```

## 전체 개발환경 docker-compose.yml (특수한 경우)

```yaml
# docker-compose.full-dev.yml - 정말 필요한 경우만
version: "3.8"

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - .:/app
      - /app/node_modules
      - /app/apps/*/node_modules
      - /app/packages/*/node_modules
    environment:
      - NODE_ENV=development
      - CHOKIDAR_USEPOLLING=true
      - NEXT_PUBLIC_API_URL=http://localhost:4000
    depends_on:
      - api

  api:
    build:
      context: .
      dockerfile: apps/api/Dockerfile.dev
    ports:
      - "4000:4000"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://postgres:postgres@postgres:5432/app_dev
    depends_on:
      - postgres

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: app_dev
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

## 프로덕션용 docker-compose.yml

```yaml
version: "3.8"

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - NEXT_PUBLIC_API_URL=${API_URL}
      - DATABASE_URL=${DATABASE_URL}
    restart: unless-stopped
    depends_on:
      - api

  api:
    build:
      context: .
      dockerfile: apps/api/Dockerfile
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
    restart: unless-stopped
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - web
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

## 환경변수 설정

### .env.local (개발용)

```bash
NEXT_PUBLIC_API_URL=http://localhost:4000
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/app_dev
```

### .env.production (프로덕션용)

```bash
NEXT_PUBLIC_API_URL=https://api.yourdomain.com
DATABASE_URL=postgresql://user:password@postgres:5432/app_prod
REDIS_URL=redis://:password@redis:6379
```

## Nginx 설정

```nginx
events {
  worker_connections 1024;
}

http {
  upstream nextjs {
    server web:3000;
  }

  server {
    listen 80;
    server_name yourdomain.com;

    location / {
      proxy_pass http://nextjs;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_cache_bypass $http_upgrade;
    }

    # 정적 파일 캐싱
    location /_next/static {
      proxy_pass http://nextjs;
      add_header Cache-Control "public, max-age=31536000, immutable";
    }
  }
}
```

## 최적화 팁

### 멀티 스테이지 빌드 최적화

```dockerfile
# 더 최적화된 Dockerfile
FROM node:18-alpine AS base
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Turbo 및 pnpm 설치
RUN npm install -g turbo
RUN corepack enable && corepack prepare pnpm@8.15.0 --activate

FROM base AS pruner
COPY . .
RUN turbo prune --scope=@project/web --docker

# 의존성만 설치
FROM base AS deps
COPY --from=pruner /app/out/json/ .
COPY --from=pruner /app/out/pnpm-lock.yaml ./pnpm-lock.yaml
RUN pnpm install --frozen-lockfile

# 빌드
FROM base AS builder
COPY --from=deps /app/ .
COPY --from=pruner /app/out/full/ .
ENV NEXT_TELEMETRY_DISABLED 1
RUN pnpm turbo build --filter=@project/web

# 실행
FROM node:18-alpine AS runner
# ... (위의 runner 스테이지와 동일)
```

## 스크립트

### package.json

```json
{
  "scripts": {
    "docker:dev": "docker-compose up",
    "docker:dev:build": "docker-compose build",
    "docker:prod": "docker-compose -f docker-compose.prod.yml up -d",
    "docker:prod:build": "docker-compose -f docker-compose.prod.yml build",
    "docker:logs": "docker-compose logs -f web",
    "docker:shell": "docker-compose exec web sh"
  }
}
```
