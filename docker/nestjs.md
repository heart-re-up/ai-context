# Nest.js Docker 설정

Nest.js API 서버를 위한 Docker 설정 가이드입니다.

## 🤔 개발환경에서 Docker가 필요한 경우

**대부분의 경우 개발환경에서는 Docker가 불필요합니다:**

###  Docker 불필요한 일반적인 개발

```bash
# 로컬에서 직접 실행하는 것이 더 효율적
pnpm install
pnpm --filter=@project/api dev
```

###  Docker가 필요한 특수한 경우

1. **팀 환경 통일**: 모든 팀원이 동일한 개발환경이 필요할 때
2. **복잡한 서비스 의존성**: DB, Redis, Kafka 등 여러 서비스가 필요할 때
3. **환경 격리**: 여러 프로젝트 간 Node.js 버전 충돌 방지
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

# API 서버 실행
EXPOSE 3000
CMD ["pnpm", "--filter=@project/api", "start:dev"]
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

# 비root 사용자 생성
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nestjs

# 프로덕션 의존성만 설치
COPY package.json ./
RUN npm ci --only=production && npm cache clean --force

# CI/CD에서 빌드된 결과물 복사
COPY --chown=nestjs:nodejs apps/api/dist ./dist

USER nestjs

EXPOSE 3000
ENV PORT 3000

CMD ["node", "dist/main"]
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

# Nest.js 빌드
RUN pnpm --filter=@project/api build

# Stage 3: 실행
FROM node:18-alpine AS runner
WORKDIR /app

ENV NODE_ENV production

# 비root 사용자 생성
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nestjs

# 프로덕션 의존성 설치
COPY --from=builder /app/apps/api/package.json ./
RUN npm ci --only=production && npm cache clean --force

# 빌드 결과물 복사
COPY --from=builder --chown=nestjs:nodejs /app/apps/api/dist ./dist

USER nestjs

EXPOSE 3000
ENV PORT 3000

CMD ["node", "dist/main"]
```

## Health Check 추가

```dockerfile
# Dockerfile에 추가
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

# curl 설치 필요
RUN apk add --no-cache curl
```

## 개발용 docker-compose.yml (의존성 서비스만)

**권장 방법: 애플리케이션은 로컬에서, 의존성만 Docker로**

```yaml
# docker-compose.dev.yml - 의존성 서비스만
version: "3.8"

services:
  # 개발용 데이터베이스
  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: api_dev
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

  # 개발용 Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  # 개발용 데이터베이스 관리 도구
  pgadmin:
    image: dpage/pgadmin4
    ports:
      - "8080:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin
    depends_on:
      - postgres

volumes:
  postgres_data:
  redis_data:
```

**사용법:**

```bash
# 의존성 서비스만 실행
docker-compose -f docker-compose.dev.yml up -d

# 애플리케이션은 로컬에서 실행
pnpm --filter=@project/api dev
```

## 전체 개발환경 docker-compose.yml (특수한 경우)

```yaml
# docker-compose.full-dev.yml - 정말 필요한 경우만
version: "3.8"

services:
  api:
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
      - DATABASE_URL=postgresql://postgres:postgres@postgres:5432/api_dev
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=dev-secret
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:15-alpine
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: api_dev
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  pgadmin:
    image: dpage/pgadmin4
    ports:
      - "8080:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin
    depends_on:
      - postgres

volumes:
  postgres_data:
  redis_data:
```

## 프로덕션용 docker-compose.yml

```yaml
version: "3.8"

services:
  api:
    build:
      context: .
      dockerfile: Dockerfile
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - JWT_SECRET=${JWT_SECRET}
      - CORS_ORIGIN=${CORS_ORIGIN}
    restart: unless-stopped
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: ${DB_NAME}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backup:/backup
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

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
      - api
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:
```

## Nest.js Health Check 설정

```typescript
// src/health/health.controller.ts
import { Controller, Get } from "@nestjs/common";
import {
  HealthCheck,
  HealthCheckService,
  TypeOrmHealthIndicator,
  MemoryHealthIndicator,
  DiskHealthIndicator,
} from "@nestjs/terminus";

@Controller("health")
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
    private memory: MemoryHealthIndicator,
    private disk: DiskHealthIndicator
  ) {}

  @Get()
  @HealthCheck()
  check() {
    return this.health.check([
      () => this.db.pingCheck("database"),
      () => this.memory.checkHeap("memory_heap", 150 * 1024 * 1024),
      () =>
        this.disk.checkStorage("storage", {
          path: "/",
          thresholdPercent: 0.9,
        }),
    ]);
  }
}
```

## 환경변수 설정

### .env (개발용)

```bash
NODE_ENV=development
PORT=3000

# Database
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/api_dev
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=postgres
DB_PASSWORD=postgres
DB_DATABASE=api_dev

# Redis
REDIS_URL=redis://localhost:6379
REDIS_HOST=localhost
REDIS_PORT=6379

# JWT
JWT_SECRET=your-dev-secret-key
JWT_EXPIRATION_TIME=24h

# CORS
CORS_ORIGIN=http://localhost:3000

# Logging
LOG_LEVEL=debug
```

### .env.production

```bash
NODE_ENV=production
PORT=3000

# Database
DATABASE_URL=${DATABASE_URL}
DB_HOST=${DB_HOST}
DB_PORT=${DB_PORT}
DB_USERNAME=${DB_USERNAME}
DB_PASSWORD=${DB_PASSWORD}
DB_DATABASE=${DB_DATABASE}

# Redis
REDIS_URL=${REDIS_URL}
REDIS_PASSWORD=${REDIS_PASSWORD}

# JWT
JWT_SECRET=${JWT_SECRET}
JWT_EXPIRATION_TIME=1h

# CORS
CORS_ORIGIN=${CORS_ORIGIN}

# Logging
LOG_LEVEL=warn
```

## TypeORM 설정

```typescript
// src/config/database.config.ts
import { TypeOrmModuleOptions } from "@nestjs/typeorm";

export const databaseConfig: TypeOrmModuleOptions = {
  type: "postgres",
  url: process.env.DATABASE_URL,
  entities: [__dirname + "/../**/*.entity{.ts,.js}"],
  synchronize: process.env.NODE_ENV === "development",
  migrations: [__dirname + "/../migrations/*{.ts,.js}"],
  migrationsRun: process.env.NODE_ENV === "production",
  logging: process.env.NODE_ENV === "development",
  ssl:
    process.env.NODE_ENV === "production"
      ? { rejectUnauthorized: false }
      : false,
};
```

## Nginx 설정

```nginx
events {
  worker_connections 1024;
}

http {
  upstream nestjs {
    server api:3000;
  }

  # Rate limiting
  limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

  server {
    listen 80;
    server_name api.yourdomain.com;

    # Rate limiting
    limit_req zone=api burst=20 nodelay;

    # Health check
    location /health {
      proxy_pass http://nestjs;
      access_log off;
    }

    # API endpoints
    location /api {
      proxy_pass http://nestjs;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_cache_bypass $http_upgrade;

      # CORS headers
      add_header 'Access-Control-Allow-Origin' '*';
      add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS';
      add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization';
    }

    # WebSocket support
    location /socket.io/ {
      proxy_pass http://nestjs;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto $scheme;
    }
  }
}
```

## 마이그레이션 스크립트

```bash
#!/bin/bash
# scripts/migrate.sh

echo "Running database migrations..."

# 컨테이너에서 마이그레이션 실행
docker-compose exec api npm run migration:run

echo "Migrations completed!"
```

## 백업 스크립트

```bash
#!/bin/bash
# scripts/backup.sh

BACKUP_DIR="./backup"
DATE=$(date +%Y%m%d_%H%M%S)

echo "Creating database backup..."

docker-compose exec postgres pg_dump -U ${DB_USER} ${DB_NAME} > ${BACKUP_DIR}/backup_${DATE}.sql

echo "Backup created: backup_${DATE}.sql"
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
    "docker:logs": "docker-compose logs -f api",
    "docker:shell": "docker-compose exec api sh",
    "docker:migrate": "./scripts/migrate.sh",
    "docker:backup": "./scripts/backup.sh"
  }
}
```
