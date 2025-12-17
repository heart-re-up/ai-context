# Prettier 설정 가이드

Prettier는 코드 포맷팅을 자동화하여 일관된 코드 스타일을 유지하는 도구입니다.

>  **모노레포에서 Prettier 설정 패키지 구현**은 [monorepo-config.md](./monorepo-config.md#prettier-설정-패키지-구현)를 참고하세요.

## 기본 설정 (prettier.config.mjs)

```javascript
/** @type {import("prettier").Config} */
export default {
  printWidth: 100, // 한 줄의 최대 길이
  tabWidth: 2, // 탭 너비 (스페이스 2개)
  useTabs: false, // 탭 대신 스페이스 사용
  semi: true, // 문장 끝에 세미콜론 추가
  singleQuote: false, // 문자열에 큰따옴표 사용
  quoteProps: "as-needed", // 필요한 경우에만 객체 속성에 따옴표 사용
  jsxSingleQuote: false, // JSX에서도 큰따옴표 사용
  trailingComma: "es5", // ES5에서 유효한 곳에 후행 쉼표 추가
  bracketSpacing: true, // 객체 리터럴의 중괄호 내부에 공백 추가
  bracketSameLine: false, // JSX 태그의 '>'를 다음 줄로
  arrowParens: "always", // 화살표 함수 매개변수를 항상 괄호로 감싸기
  endOfLine: "lf", // 줄바꿈 문자를 LF로 통일
  embeddedLanguageFormatting: "auto", // 임베디드 언어 자동 포맷팅
};
```

## 설정 옵션 설명

### 기본 포맷팅

- **printWidth**: 코드 가독성을 위해 100자로 설정
- **tabWidth & useTabs**: JavaScript 커뮤니티 표준인 스페이스 2개 들여쓰기
- **semi**: 세미콜론 사용으로 ASI(Automatic Semicolon Insertion) 문제 방지
- **endOfLine**: 줄바꿈 문자를 LF로 통일하여 크로스 플랫폼 호환성 확보

### 따옴표 설정

- **singleQuote**: false로 설정하여 JSON과의 일관성 유지
- **jsxSingleQuote**: JSX에서는 HTML 관례에 따라 큰따옴표 사용
- **quoteProps**: 객체 속성은 필요한 경우에만 따옴표 사용

### 구문 설정

- **trailingComma**: ES5 호환성 유지하면서 Git diff 최소화
- **bracketSpacing**: 객체 리터럴 가독성 향상
- **arrowParens**: 매개변수 추가/제거 시 diff 최소화

## 최적화된 설정

모던 개발 환경에서는 다음 설정을 권장합니다:

```javascript
/** @type {import("prettier").Config} */
export default {
  printWidth: 100,
  tabWidth: 2,
  useTabs: false,
  semi: true,
  singleQuote: true, // 작은따옴표 사용 권장
  quoteProps: "as-needed",
  jsxSingleQuote: false, // JSX는 HTML 관례 따라 큰따옴표
  trailingComma: "all", // 모던 환경에서는 "all" 권장
  bracketSpacing: true,
  bracketSameLine: false,
  arrowParens: "always",
  endOfLine: "lf",
  embeddedLanguageFormatting: "auto",
};
```

### singleQuote 비교

```javascript
// singleQuote: false (큰따옴표)
const name = "John";
const message = "It's working"; // 작은따옴표 포함 시 편리

// singleQuote: true (작은따옴표) - 권장
const name = "John";
const message = "It's working"; // Prettier가 자동으로 큰따옴표로 변경
```

**singleQuote: true 권장 이유:**

- JavaScript 커뮤니티 표준
- 타이핑 속도 향상 (Shift 키 불필요)
- `.json` 파일은 자동으로 큰따옴표 사용 (JSON 표준)

### trailingComma 비교

```javascript
// trailingComma: "es5" - ES5 호환
const obj = {
  name: "John",
  age: 30, //  객체에 쉼표
};

function test(
  param1,
  param2 //  함수 매개변수에는 없음
) {}

// trailingComma: "all" - 모던 환경 권장
const obj = {
  name: "John",
  age: 30, //  객체에 쉼표
};

function test(
  param1,
  param2 //  함수 매개변수에도 쉼표
) {}

// Git diff 개선 효과
// "all" 사용 시: 새 매개변수 추가 시 1줄만 변경
```

## 무시 설정 (.prettierignore)

```
# 빌드 출력물
dist/
build/
coverage/

# 라이브러리
node_modules/

# 자동 생성 파일
*.generated.ts
*.d.ts

# 특정 파일 타입
*.min.js
*.bundle.js

# 설정 파일 (필요에 따라)
package-lock.json
pnpm-lock.yaml
```

## 통합 명령어

```json
{
  "scripts": {
    "format": "prettier --check \"src/**/*.{ts,tsx,json,md}\"",
    "format:fix": "prettier --write \"src/**/*.{ts,tsx,json,md}\"",
    "format:watch": "prettier --watch \"src/**/*.{ts,tsx}\"",
    "format:staged": "prettier --write"
  }
}
```

## 에디터 통합

### VS Code 설정 (.vscode/settings.json)

```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "prettier.requireConfig": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

### 추천 확장 프로그램

```json
{
  "recommendations": ["esbenp.prettier-vscode"]
}
```

## 성능 최적화

### 캐시 활용

```bash
# 캐시와 함께 실행
prettier --write --cache src/

# 캐시 파일 위치 지정
prettier --write --cache-location .prettierçache src/
```

### 대용량 프로젝트 처리

```bash
# 병렬 처리 (많은 파일 처리 시)
find src -name "*.ts" -o -name "*.tsx" | xargs -P 4 prettier --write

# 변경된 파일만 처리 (Git 기반)
git diff --name-only --cached | grep -E '\.(ts|tsx|js|jsx)$' | xargs prettier --write
```

## 팀 협업 가이드

### 1. 설정 파일 공유

프로젝트 루트에 `prettier.config.mjs` 파일을 커밋하여 팀 전체가 동일한 설정을 사용하도록 합니다.

### 2. Pre-commit Hook 설정

```bash
# Husky + lint-staged 설치
pnpm add -D husky lint-staged
```

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx,js,jsx}": ["prettier --write"],
    "*.{json,md,css,scss}": ["prettier --write"]
  }
}
```

### 3. CI/CD 검증

```yaml
# GitHub Actions 예시
- name: Check Prettier
  run: pnpm format --check
```

## 문제 해결

### 자주 발생하는 문제

1. **설정 파일을 찾지 못함**

   ```bash
   # 설정 파일 경로 확인
   prettier --find-config-path src/index.ts
   ```

2. **특정 파일이 포맷팅되지 않음**

   ```bash
   # 파일이 무시되는지 확인
   prettier --check src/problematic-file.ts --debug-check
   ```

3. **성능 이슈**
   ```bash
   # 더 적은 파일 패턴 사용
   prettier --write "src/**/*.{ts,tsx}" --ignore-path .prettierignore
   ```

### 디버깅 명령어

```bash
# 디버그 모드로 실행
prettier --check src/ --debug-check

# 로그 레벨 설정
prettier --write src/ --log-level debug

# 설정 확인
prettier --help config
```
