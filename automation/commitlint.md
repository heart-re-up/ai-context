# Commitlint - 커밋 메시지 규칙 검증

Commitlint는 팀의 커밋 메시지 규칙을 자동으로 검증하는 도구입니다. 일관된 커밋 히스토리를 유지하는 데 도움을 줍니다.

## 설치

```bash
# commitlint CLI와 conventional config 설치
pnpm add -D @commitlint/{cli,config-conventional}
```

## 기본 설정

### commitlint.config.js

```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    // 타입 enum
    'type-enum': [
      2,
      'always',
      [
        'feat',     // 새로운 기능
        'fix',      // 버그 수정
        'docs',     // 문서 수정
        'style',    // 코드 스타일 변경 (포맷팅, 세미콜론 등)
        'refactor', // 코드 리팩토링
        'perf',     // 성능 개선
        'test',     // 테스트 추가/수정
        'build',    // 빌드 시스템 또는 외부 의존성 변경
        'ci',       // CI 설정 파일 및 스크립트 변경
        'chore',    // 기타 변경사항
        'revert'    // 커밋 되돌리기
      ]
    ],
    // 제목 최대 길이
    'header-max-length': [2, 'always', 100],
    // 제목 대소문자
    'subject-case': [
      2,
      'never',
      ['sentence-case', 'start-case', 'pascal-case', 'upper-case']
    ],
    // 본문 줄바꿈 너비
    'body-max-line-length': [2, 'always', 100],
    // 푸터 줄바꿈 너비
    'footer-max-line-length': [2, 'always', 100]
  }
};
```

## 커밋 메시지 형식

### 기본 구조

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 예시

```
feat(auth): JWT 기반 로그인 구현

- 액세스 토큰과 리프레시 토큰 발급
- 토큰 만료 시 자동 갱신 로직 추가
- 로그아웃 시 토큰 무효화 처리

Closes #123
```

## 고급 설정

### 커스텀 규칙

```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    // 커스텀 타입 추가
    'type-enum': [
      2,
      'always',
      [
        'feat', 'fix', 'docs', 'style', 'refactor',
        'perf', 'test', 'build', 'ci', 'chore', 'revert',
        'wip',      // 작업 중
        'release'   // 릴리스
      ]
    ],
    // scope 필수
    'scope-empty': [2, 'never'],
    // scope enum
    'scope-enum': [
      2,
      'always',
      [
        'api',
        'auth',
        'ui',
        'db',
        'deps',
        'config',
        'docs'
      ]
    ],
    // 제목은 소문자로 시작
    'subject-case': [2, 'always', 'lower-case'],
    // 제목 끝에 마침표 금지
    'subject-full-stop': [2, 'never', '.'],
    // 본문 필수 (breaking changes나 이슈 참조가 있을 때)
    'body-empty': [
      2,
      'never',
      ['breaking', 'closes', 'fixes']
    ]
  }
};
```

### 프로젝트별 설정

```javascript
// 모노레포용 설정
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'scope-enum': [
      2,
      'always',
      [
        // 워크스페이스별 scope
        'app-main',
        'app-admin',
        'lib-ui',
        'lib-utils',
        // 기능별 scope
        'deps',
        'release',
        'config'
      ]
    ]
  }
};
```

## Husky와 연동

```bash
# commit-msg hook 추가
echo 'npx --no -- commitlint --edit $1' > .husky/commit-msg
chmod +x .husky/commit-msg
```

## 유용한 플러그인

### @commitlint/prompt-cli

대화형 커밋 메시지 작성 도구

```bash
pnpm add -D @commitlint/prompt-cli

# package.json
{
  "scripts": {
    "commit": "commit"
  }
}
```

### commitizen

더 강력한 대화형 커밋 도구

```bash
pnpm add -D commitizen cz-conventional-changelog

# package.json
{
  "scripts": {
    "commit": "cz"
  },
  "config": {
    "commitizen": {
      "path": "cz-conventional-changelog"
    }
  }
}
```

## CI/CD 통합

### GitHub Actions

```yaml
name: Commitlint
on: [pull_request]

jobs:
  commitlint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: wagoid/commitlint-github-action@v5
```

## 문제 해결

### 잘못된 커밋 메시지 수정

```bash
# 마지막 커밋 수정
git commit --amend

# 이전 커밋들 수정
git rebase -i HEAD~3
```

### 규칙 우회 (긴급 상황)

```bash
# commitlint 무시
git commit -m "긴급 수정" --no-verify
```

## Best Practices

1. **명확한 타입**: 변경사항의 종류를 명확히 구분
2. **구체적인 설명**: 무엇을, 왜 변경했는지 설명
3. **이슈 연결**: 관련 이슈 번호 참조
4. **Breaking Changes**: 주요 변경사항은 명시적으로 표시
5. **팀 합의**: 규칙은 팀원 모두가 동의하고 따를 수 있어야 함

## 팀별 커스터마이징 예시

```javascript
// 한국어 커밋 메시지 허용
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'feat',     // 기능
        'fix',      // 버그
        'docs',     // 문서
        'style',    // 스타일
        'refactor', // 리팩토링
        'test',     // 테스트
        'chore',    // 기타
        '기능',      // feat
        '버그',      // fix
        '문서',      // docs
        '리팩토링'    // refactor
      ]
    ],
    // 한글 제목 허용
    'subject-case': [0]
  }
};
```
