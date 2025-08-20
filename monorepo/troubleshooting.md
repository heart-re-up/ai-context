# 모노레포 구성 시 발생하는 문제들과 해결방법

이 문서는 모노레포 구성 과정에서 자주 발생하는 문제들과 검증된 해결방법을 정리합니다.

##  목차

- [의존성 관련 문제](#의존성-관련-문제)
- [TypeScript 설정 문제](#typescript-설정-문제)
- [ESLint 설정 문제](#eslint-설정-문제)
- [빌드 도구 관련 문제](#빌드-도구-관련-문제)
- [React 19 관련 문제](#react-19-관련-문제)

## 의존성 관련 문제

### 1. Tailwind CSS 버전 호환성 문제

**문제**: Tailwind CSS 4.x 버전 사용 시 PostCSS 설정 오류 발생
```bash
ERR_PNPM_NO_MATCHING_VERSION  No matching version found for tailwindcss@^3.5.4
```

**해결방법**: 
- Tailwind CSS 3.x 버전 사용 (안정적)
- 또는 Tailwind CSS 4.x에 맞는 설정 변경

```json
// apps/frontend/package.json
{
  "devDependencies": {
    "tailwindcss": "^3.4.16"  // 4.x 대신 3.x 사용
  }
}
```

### 2. React 타입 의존성 누락

**문제**: UI 컴포넌트 패키지에서 React 타입을 찾을 수 없음
```bash
error TS7016: Could not find a declaration file for module 'react'
```

**해결방법**: 패키지별로 필요한 타입 의존성 명시적 설치

```json
// packages/ui-components/package.json
{
  "devDependencies": {
    "@types/react": "^19.1.6",
    "@types/react-dom": "^19.1.5",
    "react": "^19.1.0",
    "react-dom": "^19.1.0"
  },
  "peerDependencies": {
    "react": ">=18",
    "react-dom": ">=18"
  }
}
```

## TypeScript 설정 문제

### 1. 공유 TypeScript 설정의 include 경로 문제

**문제**: 상대 경로로 인한 include 경로 오류
```bash
error TS18003: No inputs were found in config file
```

**해결방법**: 각 패키지의 tsconfig.json에서 include 경로를 명시적으로 재정의

```json
// packages/ui-components/tsconfig.json
{
  "extends": "@project/typescript-config/lib",
  "compilerOptions": {
    "jsx": "react-jsx",
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.tsx", "vite-env.d.ts"],
  "exclude": ["node_modules", "dist", "dev", "**/*.test.ts", "**/*.test.tsx"]
}
```

### 2. NestJS 엔터티 클래스 초기화 문제

**문제**: TypeScript strict 모드에서 클래스 프로퍼티 초기화 오류
```bash
error TS2564: Property 'name' has no initializer and is not definitely assigned in the constructor
```

**해결방법**: Definite Assignment Assertion (`!`) 사용

```typescript
// apps/backend/src/users/entities/user.entity.ts
export class User {
  @ApiProperty({ description: '사용자 ID', example: 1 })
  id!: number;  // ! 추가

  @ApiProperty({ description: '사용자 이름', example: '홍길동' })
  name!: string;  // ! 추가

  @ApiProperty({ description: '이메일 주소', example: 'hong@example.com' })
  email!: string;  // ! 추가
}
```

## ESLint 설정 문제

### 1. 공유 ESLint 설정에서 globals import 누락

**문제**: ESLint 설정 파일에서 globals 모듈 import 누락
```javascript
// config/eslint-config/node.mjs
export default [
  ...base,
  {
    languageOptions: {
      globals: {
        ...globals.node  // globals가 정의되지 않음
      }
    }
  }
];
```

**해결방법**: 필요한 모듈을 명시적으로 import

```javascript
// config/eslint-config/node.mjs
import base from './base.mjs';
import globals from 'globals';  // 추가

export default [
  ...base,
  {
    files: ['**/*.{ts,js}'],
    languageOptions: {
      globals: {
        ...globals.node
      }
    }
  }
];
```

## 빌드 도구 관련 문제

### 1. Turbo 빌드 출력 경고

**문제**: 백엔드 빌드 시 출력 파일을 찾을 수 없다는 경고
```bash
WARNING no output files found for task @project/backend#build
```

**해결방법**: turbo.json에서 NestJS 빌드 출력 경로 수정

```json
// turbo.json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [
        "dist/**", 
        ".next/**", 
        "!.next/cache/**", 
        "build/**",
        "apps/backend/dist/**"  // NestJS 출력 경로 추가
      ]
    }
  }
}
```

## React 19 관련 문제

### 1. JSX 네임스페이스 문제

**문제**: React 19에서 JSX 타입 참조 오류
```bash
error TS2503: Cannot find namespace 'JSX'
```

**해결방법**: React.JSX 네임스페이스 사용

```typescript
// 기존
function App(): JSX.Element {
  // ...
}

// 수정
function App(): React.JSX.Element {
  // ...
}
```

### 2. React 18 타입과의 호환성 문제

**문제**: testing-library와 React 19 버전 불일치 경고
```bash
✕ unmet peer react@^18.0.0: found 19.1.1
```

**해결방법**: 
- 현재는 경고만 발생하므로 기능상 문제없음
- 향후 testing-library React 19 지원 버전으로 업데이트 필요

## 모범 사례

### 1. 단계적 문제 해결

1. **타입 검사 먼저**: `pnpm typecheck`로 TypeScript 오류 우선 해결
2. **빌드 테스트**: `pnpm build`로 빌드 오류 확인
3. **개발 서버 실행**: `pnpm dev`로 런타임 오류 확인

### 2. 의존성 관리

```bash
# 특정 패키지에만 의존성 추가
pnpm add <package-name> --filter=@project/package-name

# 워크스페이스 패키지 참조
pnpm add @project/ui-components --filter=@project/frontend
```

### 3. 설정 파일 검증

```bash
# 모든 패키지 타입 검사
pnpm typecheck

# 모든 패키지 린트 검사  
pnpm lint

# 모든 패키지 포맷 검사
pnpm format
```

## 디버깅 팁

### 1. Turbo 캐시 문제 시

```bash
# 캐시 정리
pnpm clean:cache

# 전체 정리 후 재설치
pnpm clean && pnpm install
```

### 2. TypeScript 경로 매핑 문제 시

```bash
# TypeScript 컴파일러 추적
tsc --traceResolution --noEmit
```

### 3. ESLint 설정 문제 시

```bash
# ESLint 설정 검증
npx eslint --print-config src/App.tsx
```

이러한 문제들과 해결방법을 미리 알고 있으면 모노레포 구성 시간을 크게 단축할 수 있습니다.