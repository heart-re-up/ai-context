# ai-context

프로젝트 전반에 일관된 AI 작동을 위한 컨텍스트(페르소나, 규칙, 가이드라인) 저장소입니다. 각 프로젝트에 Git 서브모듈로 추가하여 사용하세요.

## 📋 목차

- [Submodule로 연결하기](#submodule로-연결하기)
- [일상 사용법](#일상-사용법)
- [문서 구조](#문서-구조)
- [기여하기](#기여하기)

## 🔗 Submodule로 연결하기

### 1. 새 프로젝트에 추가

다른 프로젝트에서 이 저장소를 `.ai-context` 디렉토리로 연결:

```bash
# 프로젝트 루트에서 실행
cd your-project

# submodule 추가
git submodule add https://github.com/heart-re-up/ai-context.git .ai-context

# 초기 내용 다운로드
git submodule update --init --recursive

# 변경사항 커밋
git add .gitmodules .ai-context
git commit -m "feat: add ai-context submodule for AI guidelines"
```

### 2. 기존 프로젝트에 클론

이미 `.gitmodules` 파일로 submodule이 설정된 프로젝트를 클론할 때:

```bash
# 방법 1: 클론과 동시에 submodule 다운로드
git clone --recurse-submodules https://github.com/your-org/your-project.git

# 방법 2: 클론 후 submodule 초기화
git clone https://github.com/your-org/your-project.git
cd your-project
git submodule update --init --recursive
```

## 🔄 일상 사용법

### Submodule 업데이트

AI 컨텍스트 문서가 업데이트되었을 때:

```bash
# ai-context 최신 버전으로 업데이트
git submodule update --remote .ai-context

# 변경사항 확인
git status
# Changes not staged for commit:
#   modified: .ai-context (new commits)

# 업데이트 내용 커밋
git add .ai-context
git commit -m "chore: update ai-context to latest version"
git push
```

### 자동 업데이트 설정

매번 수동으로 업데이트하기 번거롭다면:

```bash
# .gitmodules 파일 수정
[submodule ".ai-context"]
    path = .ai-context
    url = https://github.com/your-org/ai-context.git
    branch = main  # 특정 브랜치 추적

# 이후 업데이트는 간단하게
git submodule update --remote
```

## ✏️ AI Context 문서 개선하기

맥락 문서에서 오타를 발견, 또는 개선사항을 기여하는 경우에 서브모듈 컨텐츠가 변경됩니다.

### 서브모듈 내용 변경 시 올바른 Git 작업 흐름

서브모듈의 변경사항을 Git 저장소에 반영하려면, 반드시 서브모듈 디렉토리에서 Git 작업을 수행해야 합니다.

#### 베스트 프랙티스 워크플로우

프로젝트 개발 중 ai-context 문서에 오타나 개선사항을 발견했을 때:

```bash
# 1. 문서 편집 (어디서든 가능)
vim .ai-context/README.md  # 또는 code .ai-context/README.md

# 2. 서브모듈로 이동하여 Git 작업 시작
cd .ai-context

# 3. 브랜치 생성
git checkout -b fix/improve-documentation

# 4. 변경사항 확인 및 커밋
git diff                    # 변경사항 확인
git add .
git commit -m "docs: improve submodule workflow explanation"

# 5. 원격 저장소에 푸시
git push origin fix/improve-documentation

# 6. GitHub에서 Pull Request 생성
# https://github.com/heart-re-up/ai-context/compare/fix/improve-documentation
```

#### PR 승인 후 최신 버전 적용

```bash
# PR이 승인되어 main 브랜치에 머지된 후

# 1. 서브모듈을 최신 main으로 업데이트
cd .ai-context
git checkout main           # main 브랜치로 스위치
git pull origin main        # 최신 변경사항 받기

# 또는 메인 프로젝트에서 한 번에
cd ..
git submodule update --remote .ai-context

# 2. (선택사항) 팀 협업 시 메인 프로젝트에 반영
git add .ai-context
git commit -m "chore: update ai-context to latest version"
```

### 🚨 주의사항

1. **Git 작업은 반드시 서브모듈 디렉토리에서**
   - 파일 편집은 어디서든 가능
   - `git add`, `git commit`, `git push` 등은 `.ai-context/` 디렉토리에서만

2. **브랜치 상태 확인**

   ```bash
   # submodule의 현재 상태 확인
   git submodule status

   # detached HEAD 상태라면 브랜치로 체크아웃
   cd .ai-context
   git checkout main
   ```

## 🛠️ 문제 해결

### 자주 발생하는 문제

1. **Submodule이 비어있을 때**

   ```bash
   git submodule update --init --recursive
   ```

2. **Submodule 변경사항이 반영되지 않을 때**

   ```bash
   # 강제 업데이트
   git submodule update --remote --force .ai-context
   ```

3. **Submodule 제거하고 싶을 때**

   ```bash
   # submodule 제거
   git submodule deinit .ai-context
   git rm .ai-context
   git commit -m "remove ai-context submodule"
   ```

4. **다른 브랜치로 전환 후 submodule 문제**
   ```bash
   # submodule 상태 동기화
   git submodule update --recursive
   ```

## 📁 문서 구조

```
.ai-context/
├── README.md              # 이 파일
├── automation/            # 자동화 도구 설정
├── cicd/                  # CI/CD 파이프라인 가이드
├── docker/                # Docker 설정 가이드
├── monorepo/              # 모노레포 구성 가이드
├── node/                  # Node.js 프로젝트 설정
├── quality/               # 코드 품질 관리
├── typescript.md          # TypeScript 설정 가이드
├── vite/                  # Vite 빌드 도구 설정
├── vitest/                # Vitest 테스트 설정
└── vscode/                # VS Code 설정
```

## 🤝 기여하기

### 새 가이드라인 추가

1. 이슈 생성으로 제안사항 논의
2. 브랜치 생성 후 문서 작성
3. Pull Request로 리뷰 요청
4. 승인 후 main 브랜치에 병합

### 기존 문서 개선

1. 오타나 개선사항 발견시 즉시 수정 가능
2. 큰 변경사항은 이슈로 먼저 논의
3. 실제 프로젝트 적용 경험을 바탕으로 업데이트

## 💡 활용 팁

### AI 어시스턴트와 함께 사용

이 문서들을 AI 어시스턴트의 컨텍스트로 제공하여:

- 일관된 코딩 스타일 유지
- 프로젝트 표준에 맞는 설정 생성
- 모범 사례 기반 코드 리뷰

### 팀 온보딩

새 팀원에게는 이 저장소 링크만 공유하면 됩니다:

📖 **프로젝트 가이드라인**: `.ai-context/` 디렉토리 참고  
🔗 **원본 저장소**: https://github.com/your-org/ai-context

주요 문서들이 `.ai-context/` 디렉토리에 정리되어 있습니다.
