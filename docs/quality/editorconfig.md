# EditorConfig 설정 가이드

EditorConfig는 다양한 에디터와 IDE에서 일관된 코딩 스타일을 유지하기 위한 설정 파일입니다.

## 기본 설정 (.editorconfig)

```ini
# .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 2
insert_final_newline = true
trim_trailing_whitespace = true

[*.md]
trim_trailing_whitespace = false

[*.{yml,yaml}]
indent_size = 2

[*.{js,ts,jsx,tsx}]
indent_size = 2

[*.json]
indent_size = 2

[*.{py,rb}]
indent_size = 4

[Makefile]
indent_style = tab
```

## 설정 옵션 설명

### 전역 설정 (`[*]`)

- **charset**: 파일 인코딩을 UTF-8로 통일
- **end_of_line**: 줄바꿈 문자를 LF로 설정 (크로스 플랫폼 호환성)
- **indent_style**: 들여쓰기 방식 (space 또는 tab)
- **indent_size**: 들여쓰기 크기
- **insert_final_newline**: 파일 끝에 빈 줄 추가
- **trim_trailing_whitespace**: 줄 끝 공백 제거

### 파일별 특화 설정

#### 마크다운 파일 (`[*.md]`)

```ini
[*.md]
trim_trailing_whitespace = false
```

- 마크다운에서는 줄 끝 공백이 의미가 있을 수 있으므로 제거하지 않음

#### YAML 파일 (`[*.{yml,yaml}]`)

```ini
[*.{yml,yaml}]
indent_size = 2
```

- YAML은 들여쓰기가 중요하므로 명시적으로 2칸 설정

#### 웹 관련 파일 (`[*.{js,ts,jsx,tsx,json}]`)

```ini
[*.{js,ts,jsx,tsx}]
indent_size = 2

[*.json]
indent_size = 2
```

- JavaScript/TypeScript 표준인 2칸 들여쓰기

#### 백엔드 언어 (`[*.{py,rb}]`)

```ini
[*.{py,rb}]
indent_size = 4
```

- Python, Ruby는 관례적으로 4칸 들여쓰기

#### Makefile (`[Makefile]`)

```ini
[Makefile]
indent_style = tab
```

- Makefile은 반드시 탭 문자 사용

## 고급 설정

### 프로젝트별 맞춤 설정

```ini
# .editorconfig for a React TypeScript project
root = true

# 기본 설정
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

# 웹 프론트엔드 파일
[*.{js,jsx,ts,tsx,vue,svelte}]
indent_style = space
indent_size = 2

# 스타일시트
[*.{css,scss,sass,less}]
indent_style = space
indent_size = 2

# 설정 파일
[*.{json,jsonc,json5}]
indent_style = space
indent_size = 2

[*.{yml,yaml}]
indent_style = space
indent_size = 2

# 마크다운
[*.md]
indent_style = space
indent_size = 2
trim_trailing_whitespace = false

# 템플릿 파일
[*.{html,xml,svg}]
indent_style = space
indent_size = 2

# 쉘 스크립트
[*.{sh,bash,zsh}]
indent_style = space
indent_size = 2

# Docker
[{Dockerfile,Dockerfile.*}]
indent_style = space
indent_size = 2

# Git
[.gitcommit]
max_line_length = 72
```

### 팀 표준 설정

```ini
# 엔터프라이즈 프로젝트용 설정
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
max_line_length = 100

# 프론트엔드
[*.{js,jsx,ts,tsx}]
indent_style = space
indent_size = 2
max_line_length = 100

# 백엔드 (Node.js)
[*.{js,ts}]
indent_style = space
indent_size = 2

# 백엔드 (Java)
[*.java]
indent_style = space
indent_size = 4
max_line_length = 120

# 백엔드 (Python)
[*.py]
indent_style = space
indent_size = 4
max_line_length = 88

# 데이터베이스
[*.sql]
indent_style = space
indent_size = 2

# 인프라
[*.{tf,hcl}]
indent_style = space
indent_size = 2

[*.{yml,yaml}]
indent_style = space
indent_size = 2

# 문서
[*.{md,rst}]
indent_style = space
indent_size = 2
trim_trailing_whitespace = false
max_line_length = 80
```

## 에디터별 지원

### VS Code

- **기본 지원**: 별도 확장 프로그램 불필요
- **설정 연동**: EditorConfig 설정이 VS Code 설정보다 우선

### IntelliJ IDEA / WebStorm

- **기본 지원**: 2017.1 버전부터 기본 지원
- **플러그인**: 이전 버전은 EditorConfig 플러그인 설치 필요

### Sublime Text

- **플러그인 필요**: EditorConfig 패키지 설치

### Vim/Neovim

- **플러그인**: editorconfig-vim 플러그인 설치

### Atom

- **기본 지원**: editorconfig 패키지가 코어에 포함

## 다른 도구와의 통합

### Prettier와의 관계

```ini
# .editorconfig
[*.{js,ts,jsx,tsx}]
indent_style = space
indent_size = 2
end_of_line = lf
```

```javascript
// prettier.config.mjs
export default {
  tabWidth: 2, // EditorConfig의 indent_size와 일치
  useTabs: false, // EditorConfig의 indent_style과 일치
  endOfLine: "lf", // EditorConfig의 end_of_line과 일치
};
```

### ESLint와의 관계

```javascript
// eslint.config.mjs
export default [
  {
    rules: {
      // EditorConfig 설정과 충돌하지 않도록 조정
      indent: ["error", 2], // EditorConfig indent_size와 일치
      "linebreak-style": ["error", "unix"], // EditorConfig end_of_line과 일치
      "eol-last": ["error", "always"], // EditorConfig insert_final_newline과 일치
    },
  },
];
```

## 모노레포 설정

### 루트 설정

```ini
# 루트 .editorconfig
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

# 공통 웹 파일
[*.{js,ts,jsx,tsx,json,css,scss,html}]
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

### 패키지별 재정의

```ini
# packages/mobile/.editorconfig
root = true

# 모바일 개발은 4칸 들여쓰기 선호
[*.{js,ts,jsx,tsx}]
indent_style = space
indent_size = 4

[*.json]
indent_style = space
indent_size = 2
```

## 검증 및 테스트

### 설정 확인 도구

```bash
# EditorConfig 설정 확인 (Node.js)
npx editorconfig-checker

# 특정 파일의 설정 확인
npx editorconfig --version
```

### CI/CD 통합

```yaml
# GitHub Actions
name: EditorConfig Check
on: [push, pull_request]

jobs:
  editorconfig:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Check EditorConfig
        uses: editorconfig-checker/action-editorconfig-checker@main
        with:
          exclude: |
            node_modules/
            dist/
            build/
```

### Pre-commit Hook

```bash
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/editorconfig-checker/editorconfig-checker.python
    rev: main
    hooks:
      - id: editorconfig-checker
```

## 문제 해결

### 자주 발생하는 문제

1. **설정이 적용되지 않음**

   - 에디터 재시작 필요
   - 파일이 이미 열려있는 경우 다시 열기
   - 에디터의 EditorConfig 지원 확인

2. **다른 도구와 충돌**

   - Prettier/ESLint 설정과 일치하는지 확인
   - 우선순위: EditorConfig > Prettier > ESLint

3. **파일별 설정 오류**
   - 패턴 매칭 확인: `[*.{js,ts}]` vs `[*.js,*.ts]`
   - 대소문자 구분 확인

### 디버깅 방법

```bash
# 설정 유효성 검사
editorconfig-checker

# 특정 파일에 적용된 설정 확인
editorconfig src/index.ts
```

## 모범 사례

### 1. 프로젝트 시작 시 설정

```ini
# 새 프로젝트용 최소 설정
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

[*.{js,ts,jsx,tsx,json,css,html,md}]
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

### 2. 팀 도입 전략

1. **1단계**: 기본 설정 (charset, end_of_line, insert_final_newline)
2. **2단계**: 들여쓰기 설정 (indent_style, indent_size)
3. **3단계**: 세부 설정 (max_line_length, 파일별 특화)

### 3. 유지보수

- 정기적인 설정 검토 및 업데이트
- 새로운 파일 타입 추가 시 설정 추가
- 팀 피드백을 통한 설정 개선

## 참고 자료

- [EditorConfig 공식 사이트](https://editorconfig.org/)
- [지원 에디터 목록](https://editorconfig.org/#download)
- [속성 설명서](https://editorconfig.org/#supported-properties)
