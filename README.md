# ai-context

프로젝트 전반에 일관된 AI 작동을 위한 컨텍스트(페르소나, 규칙, 가이드라인) 저장소입니다. 각 프로젝트에 Git 서브모듈로 추가하여 사용하세요.

## 목차

- [Submodule로 연결하기](#submodule로-연결하기)
- [일상 사용법](#일상-사용법)
- [문서 구조](#문서-구조)
- [기여하기](#기여하기)

## Submodule로 연결하기

### 1. 새 프로젝트에 추가

다른 프로젝트에서 이 저장소를 `.ai-context` 디렉토리로 연결:

```bash
# 프로젝트 루트에서 실행
cd your-project

# submodule 추가
git submodule add https://github.com/your-org/ai-context.git .ai-context

# 초기 내용 다운로드
git submodule update --init --recursive

# 변경사항 커밋
git add .gitmodules .ai-context
git commit -m "feat: add ai-context submodule for AI guidelines"
```

### 2. 기존 프로젝트에 클론

이미 submodule이 설정된 프로젝트를 클론할 때:

```bash
# 방법 1: 클론과 동시에 submodule 다운로드
git clone --recurse-submodules https://github.com/your-org/your-project.git

# 방법 2: 클론 후 submodule 초기화
git clone https://github.com/your-org/your-project.git
cd your-project
git submodule update --init --recursive
```

## 일상 사용법

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

## AI Context 수정하기

### 문서 수정 및 기여

AI 컨텍스트 문서를 개선하고 싶을 때:

```bash
# 1. ai-context 디렉토리로 이동
cd .ai-context

# 2. 새 브랜치 생성
git checkout -b improve/add-new-guideline

# 3. 문서 수정
# (에디터에서 필요한 문서 수정)

# 4. 변경사항 커밋
git add .
git commit -m "docs: add new coding guideline for API design"

# 5. 원격 저장소에 푸시
git push origin improve/add-new-guideline

# 6. GitHub에서 Pull Request 생성
# https://github.com/your-org/ai-context/compare/improve/add-new-guideline

# 7. 메인 프로젝트로 돌아가기
cd ..

# 8. 수정된 내용을 메인 프로젝트에 반영 (PR 머지 후)
git submodule update --remote .ai-context
git add .ai-context
git commit -m "chore: update ai-context with new guidelines"
```

### 긴급 수정 (직접 커밋)

간단한 수정이나 오타 수정의 경우:

```bash
# ai-context 디렉토리에서 직접 작업
cd .ai-context

# 파일 수정 후 즉시 커밋
git add .
git commit -m "fix: correct typo in coding standards"
git push origin main

# 메인 프로젝트에 반영
cd ..
git submodule update --remote .ai-context
git add .ai-context
git commit -m "chore: update ai-context with typo fixes"
git push
```

## 주의사항

### Submodule 작업시 주의점

1. **항상 올바른 디렉토리에서 작업**
   ```bash
   # 잘못된 예: 메인 프로젝트에서 ai-context 파일 수정
   vim .ai-context/README.md  # 변경사항이 추적되지 않음
   
   # 올바른 예: ai-context 디렉토리에서 작업
   cd .ai-context
   vim README.md
   git add README.md
   ```

2. **커밋 순서 중요**
   ```bash
   # 1순위: ai-context 저장소에 커밋
   cd .ai-context
   git commit -m "docs: update guidelines"
   git push
   
   # 2순위: 메인 프로젝트에 submodule 업데이트 반영
   cd ..
   git add .ai-context
   git commit -m "chore: update ai-context"
   ```

3. **브랜치 상태 확인**
   ```bash
   # submodule의 현재 상태 확인
   git submodule status
   
   # detached HEAD 상태라면 브랜치로 체크아웃
   cd .ai-context
   git checkout main
   ```

## 문제 해결

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

## 문서 구조

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

## 기여하기

### 새 가이드라인 추가

1. 이슈 생성으로 제안사항 논의
2. 브랜치 생성 후 문서 작성
3. Pull Request로 리뷰 요청
4. 승인 후 main 브랜치에 병합

### 기존 문서 개선

1. 오타나 개선사항 발견시 즉시 수정 가능
2. 큰 변경사항은 이슈로 먼저 논의
3. 실제 프로젝트 적용 경험을 바탕으로 업데이트

## 활용 팁

### AI 어시스턴트와 함께 사용

이 문서들을 AI 어시스턴트의 컨텍스트로 제공하여:

- 일관된 코딩 스타일 유지
- 프로젝트 표준에 맞는 설정 생성
- 모범 사례 기반 코드 리뷰

### 팀 온보딩

새 팀원에게는 이 저장소 링크만 공유하면 됩니다:

**프로젝트 가이드라인**: `.ai-context/` 디렉토리 참고  
**원본 저장소**: https://github.com/your-org/ai-context

주요 문서들이 `.ai-context/` 디렉토리에 정리되어 있습니다.
