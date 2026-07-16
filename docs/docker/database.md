# 데이터베이스 Docker 설정

다양한 데이터베이스를 위한 Docker 설정 가이드입니다.

## PostgreSQL

### 기본 설정

```yaml
# docker-compose.yml
version: "3.8"

services:
  postgres:
    image: postgres:15-alpine
    container_name: postgres_db
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}
      POSTGRES_DB: ${DB_NAME:-myapp}
      POSTGRES_INITDB_ARGS: "--encoding=UTF-8 --locale=C"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init:/docker-entrypoint-initdb.d
      - ./backup:/backup
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5

  # PostgreSQL 관리 도구
  pgadmin:
    image: dpage/pgadmin4
    container_name: pgadmin
    ports:
      - "8080:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin123
    volumes:
      - pgadmin_data:/var/lib/pgadmin
    depends_on:
      - postgres

volumes:
  postgres_data:
  pgadmin_data:
```

### 초기화 스크립트

```sql
-- init/01-init.sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- 사용자 테이블
CREATE TABLE IF NOT EXISTS users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(100) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 인덱스 생성
CREATE INDEX IF NOT EXISTS idx_users_email ON users(email);
CREATE INDEX IF NOT EXISTS idx_users_created_at ON users(created_at);
```

## MongoDB

```yaml
# docker-compose.yml
version: "3.8"

services:
  mongodb:
    image: mongo:6.0
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_ROOT_USER:-admin}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD:-password}
      MONGO_INITDB_DATABASE: ${MONGO_DB:-myapp}
    volumes:
      - mongodb_data:/data/db
      - ./mongo-init:/docker-entrypoint-initdb.d
    restart: unless-stopped

  # MongoDB 관리 도구
  mongo-express:
    image: mongo-express
    container_name: mongo_express
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: ${MONGO_ROOT_USER:-admin}
      ME_CONFIG_MONGODB_ADMINPASSWORD: ${MONGO_ROOT_PASSWORD:-password}
      ME_CONFIG_MONGODB_URL: mongodb://${MONGO_ROOT_USER:-admin}:${MONGO_ROOT_PASSWORD:-password}@mongodb:27017/
    depends_on:
      - mongodb

volumes:
  mongodb_data:
```

## Redis

```yaml
# docker-compose.yml
version: "3.8"

services:
  redis:
    image: redis:7-alpine
    container_name: redis_cache
    ports:
      - "6379:6379"
    command: redis-server --requirepass ${REDIS_PASSWORD:-redis123} --appendonly yes
    volumes:
      - redis_data:/data
      - ./redis.conf:/usr/local/etc/redis/redis.conf
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis 관리 도구
  redis-commander:
    image: rediscommander/redis-commander:latest
    container_name: redis_commander
    ports:
      - "8082:8081"
    environment:
      REDIS_HOSTS: local:redis:6379:0:${REDIS_PASSWORD:-redis123}
    depends_on:
      - redis

volumes:
  redis_data:
```

### Redis 설정 파일

```conf
# redis.conf
# 메모리 설정
maxmemory 256mb
maxmemory-policy allkeys-lru

# 영속성 설정
save 900 1
save 300 10
save 60 10000

# 로그 설정
loglevel notice
logfile /var/log/redis/redis-server.log

# 네트워크 설정
timeout 300
tcp-keepalive 300
```

## MySQL

```yaml
# docker-compose.yml
version: "3.8"

services:
  mysql:
    image: mysql:8.0
    container_name: mysql_db
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-rootpassword}
      MYSQL_DATABASE: ${MYSQL_DATABASE:-myapp}
      MYSQL_USER: ${MYSQL_USER:-user}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD:-password}
    volumes:
      - mysql_data:/var/lib/mysql
      - ./mysql-init:/docker-entrypoint-initdb.d
    command: --default-authentication-plugin=mysql_native_password
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  # MySQL 관리 도구
  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: phpmyadmin
    ports:
      - "8083:80"
    environment:
      PMA_HOST: mysql
      PMA_PORT: 3306
      PMA_USER: root
      PMA_PASSWORD: ${MYSQL_ROOT_PASSWORD:-rootpassword}
    depends_on:
      - mysql

volumes:
  mysql_data:
```

## 전체 스택 예제

```yaml
# docker-compose.full.yml
version: "3.8"

services:
  # 웹 애플리케이션
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://postgres:postgres@postgres:5432/myapp
      - REDIS_URL=redis://:redis123@redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  # PostgreSQL
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis
  redis:
    image: redis:7-alpine
    command: redis-server --requirepass redis123
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "--raw", "incr", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  # MongoDB (선택사항)
  mongodb:
    image: mongo:6.0
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password
    volumes:
      - mongodb_data:/data/db

  # 관리 도구들
  pgadmin:
    image: dpage/pgadmin4
    ports:
      - "8080:80"
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin
    depends_on:
      - postgres

  redis-commander:
    image: rediscommander/redis-commander:latest
    ports:
      - "8081:8081"
    environment:
      REDIS_HOSTS: local:redis:6379:0:redis123
    depends_on:
      - redis

volumes:
  postgres_data:
  redis_data:
  mongodb_data:
```

## 백업 및 복원

### PostgreSQL 백업

```bash
#!/bin/bash
# scripts/postgres-backup.sh

CONTAINER_NAME="postgres_db"
DB_NAME="myapp"
BACKUP_DIR="./backup"
DATE=$(date +%Y%m%d_%H%M%S)

# 백업 생성
docker exec $CONTAINER_NAME pg_dump -U postgres $DB_NAME > $BACKUP_DIR/postgres_backup_$DATE.sql

echo "PostgreSQL backup created: postgres_backup_$DATE.sql"
```

### PostgreSQL 복원

```bash
#!/bin/bash
# scripts/postgres-restore.sh

CONTAINER_NAME="postgres_db"
DB_NAME="myapp"
BACKUP_FILE=$1

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup_file.sql>"
    exit 1
fi

# 복원
docker exec -i $CONTAINER_NAME psql -U postgres $DB_NAME < $BACKUP_FILE

echo "PostgreSQL restore completed from: $BACKUP_FILE"
```

### MongoDB 백업

```bash
#!/bin/bash
# scripts/mongodb-backup.sh

CONTAINER_NAME="mongodb"
DB_NAME="myapp"
BACKUP_DIR="./backup"
DATE=$(date +%Y%m%d_%H%M%S)

# 백업 생성
docker exec $CONTAINER_NAME mongodump --db $DB_NAME --out /tmp/backup
docker cp $CONTAINER_NAME:/tmp/backup $BACKUP_DIR/mongodb_backup_$DATE

echo "MongoDB backup created: mongodb_backup_$DATE"
```

## 모니터링

### 데이터베이스 상태 확인

```bash
#!/bin/bash
# scripts/db-status.sh

echo "=== PostgreSQL Status ==="
docker exec postgres_db pg_isready -U postgres

echo "=== Redis Status ==="
docker exec redis_cache redis-cli ping

echo "=== MongoDB Status ==="
docker exec mongodb mongo --eval "db.adminCommand('ping')"

echo "=== Container Status ==="
docker-compose ps
```

## 환경변수 파일

### .env.example

```bash
# PostgreSQL
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=myapp

# Redis
REDIS_PASSWORD=redis123

# MySQL
MYSQL_ROOT_PASSWORD=rootpassword
MYSQL_DATABASE=myapp
MYSQL_USER=user
MYSQL_PASSWORD=password

# MongoDB
MONGO_ROOT_USER=admin
MONGO_ROOT_PASSWORD=password
MONGO_DB=myapp
```

## 성능 최적화

### PostgreSQL 최적화

```sql
-- postgresql.conf 설정
shared_buffers = 256MB
effective_cache_size = 1GB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
```

### Redis 최적화

```conf
# redis.conf 최적화
maxmemory-policy allkeys-lru
save 900 1
save 300 10
save 60 10000
tcp-keepalive 300
timeout 300
```
