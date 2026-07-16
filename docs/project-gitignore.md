# Git 무시 설정 (.gitignore)

프로젝트에서 버전 관리에서 제외할 파일 및 디렉토리 설정입니다.

## IDE 관련 파일

```gitignore
# VS Code
.vscode/*
!.vscode/settings.json    # 워크스페이스 설정은 공유
!.vscode/tasks.json       # 태스크 설정은 공유
!.vscode/launch.json      # 디버그 설정은 공유
!.vscode/extensions.json  # 확장 프로그램 추천은 공유

# IntelliJ IDEA / WebStorm
.idea/                    # IntelliJ 설정 파일들
*.iml                     # IntelliJ 모듈 파일
*.iws                     # IntelliJ 워크스페이스 파일
*.ipr                     # IntelliJ 프로젝트 파일
```

## Node.js 관련 파일

```gitignore
# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# Lock files (프로젝트에 따라 선택)
# package-lock.json       # npm 사용 시
# yarn.lock              # yarn 사용 시
# pnpm-lock.yaml         # pnpm 사용 시 (보통 포함)
```

## 빌드 및 배포 파일

```gitignore
# Build outputs
dist/
build/
.next/
.nuxt/
.output/

# Temporary files
.tmp/
.cache/
.parcel-cache/

# Environment files
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
```

## 운영체제별 파일

```gitignore
# macOS
.DS_Store
.AppleDouble
.LSOverride
Icon

# Windows
Thumbs.db
Thumbs.db:encryptable
ehthumbs.db
ehthumbs_vista.db
*.tmp
Desktop.ini

# Linux
*~
.fuse_hidden*
.directory
.Trash-*
```

## 로그 및 런타임 파일

```gitignore
# Logs
logs/
*.log
lerna-debug.log*

# Runtime data
pids/
*.pid
*.seed
*.pid.lock

# Coverage directory used by tools like istanbul
coverage/
*.lcov

# Test results
test-results/
```

## 전체 .gitignore 예시

```gitignore
# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# Build outputs
dist/
build/
.next/
.nuxt/
.output/
.tmp/
.cache/
.parcel-cache/

# Environment files
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# IDE
.vscode/*
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json
.idea/
*.iml
*.iws
*.ipr

# OS generated files
.DS_Store
.AppleDouble
.LSOverride
Icon
Thumbs.db
Thumbs.db:encryptable
ehthumbs.db
ehthumbs_vista.db
*.tmp
Desktop.ini
*~
.fuse_hidden*
.directory
.Trash-*

# Logs
logs/
*.log
lerna-debug.log*

# Coverage
coverage/
*.lcov

# Test results
test-results/
```

## 주의사항

1. **Lock 파일**: 프로젝트 팩키지 매니저에 따라 해당하는 lock 파일은 포함해야 합니다.
2. **환경 변수**: `.env` 파일은 보안상 무시하되, `.env.example` 같은 템플릿은 포함하세요.
3. **IDE 설정**: 팀에서 공유해야 하는 설정은 선택적으로 포함합니다.
