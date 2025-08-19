# Lint-staged - Git Staged 파일 검증

Lint-staged는 Git에 staged된 파일만 선택적으로 검사하여 커밋 시간을 단축시키는 도구입니다.

## 설치

```bash
pnpm add -D lint-staged
```

## 기본 설정

### package.json 방식

```json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{css,scss,less}": [
      "stylelint --fix",
      "prettier --write"
    ],
    "*.{json,md,yml,yaml}": [
      "prettier --write"
    ]
  }
}
```

### .lintstagedrc.js 방식

```javascript
module.exports = {
  // JavaScript/TypeScript 파일
  '*.{js,jsx,ts,tsx}': [
    'eslint --fix',
    'prettier --write',
    'vitest related --run'
  ],
  // 스타일 파일
  '*.{css,scss,less}': [
    'stylelint --fix',
    'prettier --write'
  ],
  // JSON, Markdown, YAML
  '*.{json,md,yml,yaml}': [
    'prettier --write'
  ],
  // 이미지 최적화
  '*.{png,jpeg,jpg,gif,svg}': [
    'imagemin-lint-staged'
  ]
};
```

## 고급 설정

### 함수 기반 설정

```javascript
module.exports = {
  '*.{ts,tsx}': (filenames) => {
    // 파일 수가 많으면 병렬 처리
    const commands = [];
    
    // ESLint
    if (filenames.length > 10) {
      commands.push(`eslint --fix ${filenames.join(' ')} --max-warnings 0`);
    } else {
      commands.push(...filenames.map(f => `eslint --fix ${f} --max-warnings 0`));
    }
    
    // Prettier
    commands.push(`prettier --write ${filenames.join(' ')}`);
    
    // 타입 체크 (전체 프로젝트)
    commands.push('tsc --noEmit');
    
    return commands;
  }
};
```

### 조건부 실행

```javascript
module.exports = {
  '*.{ts,tsx}': async (filenames) => {
    const commands = [];
    
    // src 디렉토리의 파일만 테스트 실행
    const srcFiles = filenames.filter(file => file.includes('/src/'));
    if (srcFiles.length > 0) {
      commands.push('pnpm test:unit');
    }
    
    // 모든 파일에 대해 lint와 format
    commands.push(`eslint --fix ${filenames.join(' ')}`);
    commands.push(`prettier --write ${filenames.join(' ')}`);
    
    return commands;
  }
};
```

## 모노레포 설정

### 루트 레벨 설정

```javascript
// .lintstagedrc.js (root)
module.exports = {
  // 각 워크스페이스별 처리
  'apps/*/src/**/*.{ts,tsx}': (filenames) => {
    const commands = [];
    
    // 워크스페이스별로 그룹화
    const filesByWorkspace = {};
    filenames.forEach(file => {
      const workspace = file.split('/')[1];
      if (!filesByWorkspace[workspace]) {
        filesByWorkspace[workspace] = [];
      }
      filesByWorkspace[workspace].push(file);
    });
    
    // 각 워크스페이스에서 명령 실행
    Object.entries(filesByWorkspace).forEach(([workspace, files]) => {
      commands.push(`pnpm --filter=@project/${workspace} lint`);
      commands.push(`pnpm --filter=@project/${workspace} test:unit`);
    });
    
    return commands;
  },
  
  // 공통 패키지
  'packages/*/src/**/*.{ts,tsx}': [
    'eslint --fix',
    'prettier --write',
    'pnpm test:unit'
  ]
};
```

### 워크스페이스별 설정

```javascript
// apps/main-app/.lintstagedrc.js
module.exports = {
  '*.{ts,tsx}': [
    'eslint --fix',
    'prettier --write',
    () => 'pnpm test:unit' // 전체 단위 테스트
  ],
  '*.module.css': [
    'stylelint --fix'
  ]
};
```

## 성능 최적화

### 병렬 실행

```javascript
module.exports = {
  // 동시에 실행 가능한 작업들
  '*.{ts,tsx}': [
    'eslint --fix',
    'prettier --write'
  ],
  // 순차 실행이 필요한 작업
  '*.spec.{ts,tsx}': [
    'eslint --fix',
    'prettier --write',
    () => 'pnpm test:unit --run' // 모든 파일 처리 후 실행
  ]
};
```

### 캐싱 활용

```javascript
module.exports = {
  '*.{ts,tsx}': [
    'eslint --cache --fix',
    'prettier --write --cache'
  ]
};
```

## Husky와 연동

```bash
# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm exec lint-staged
```

## 디버깅

### 상세 로그 출력

```bash
# 디버그 모드로 실행
DEBUG=lint-staged* git commit -m "test"

# 또는 package.json에 설정
{
  "scripts": {
    "pre-commit": "lint-staged --debug"
  }
}
```

### Dry run

```bash
# 실제 실행하지 않고 명령만 확인
pnpm exec lint-staged --dry-run
```

## 문제 해결

### 느린 실행 속도

```javascript
module.exports = {
  // 타입 체크는 전체 프로젝트에 한 번만
  '*.{ts,tsx}': (filenames) => [
    `eslint --fix ${filenames.join(' ')}`,
    `prettier --write ${filenames.join(' ')}`,
    'tsc --noEmit' // 파일별로 실행하지 않음
  ]
};
```

### 파일 경로 이슈

```javascript
module.exports = {
  // 상대 경로 사용
  '*.{ts,tsx}': (filenames) => 
    filenames.map(file => `eslint --fix "${file}"`)
};
```

## Best Practices

1. **빠른 작업 우선**: 빠른 작업(prettier)을 먼저, 느린 작업(test)을 나중에
2. **필수 작업만**: 커밋을 막을 정도로 중요한 검사만 포함
3. **점진적 적용**: 처음엔 간단한 규칙부터 시작
4. **명확한 에러**: 실패 이유를 명확히 표시
5. **캐시 활용**: 가능한 도구들은 캐시 옵션 사용

## 다른 도구와의 통합

```javascript
module.exports = {
  // Prettier + ESLint
  '*.{js,jsx,ts,tsx}': [
    'prettier --write',
    'eslint --fix'
  ],
  
  // Stylelint
  '*.{css,scss}': [
    'stylelint --fix',
    'prettier --write'
  ],
  
  // Markdown
  '*.md': [
    'markdownlint --fix',
    'prettier --write'
  ],
  
  // 커밋 메시지용 (commit-msg hook에서 사용)
  '.commitlintrc.js': [
    'prettier --write'
  ]
};
```
