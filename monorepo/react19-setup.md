# React 19 모노레포 설정 가이드

React 19를 사용한 모노레포 구성 시 주의사항과 설정 방법을 정리합니다.

## React 19 주요 변경사항

### 1. JSX 타입 변경

React 19에서 JSX 타입 참조 방식이 변경되었습니다.

```typescript
//  React 18 방식 (더 이상 작동하지 않음)
function App(): JSX.Element {
  return <div>Hello</div>;
}

//  React 19 방식
function App(): React.JSX.Element {
  return <div>Hello</div>;
}

// 또는 간단히
function App() {
  return <div>Hello</div>;
}
```

### 2. TypeScript 설정

React 19에 맞는 TypeScript 설정:

```json
// config/typescript-config/app.json
{
  "compilerOptions": {
    "target": "ES2016",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "jsx": "react-jsx",  // react-jsx 사용
    "useDefineForClassFields": true,
    "strict": true
  }
}
```

### 3. 의존성 버전

```json
// package.json
{
  "dependencies": {
    "react": "^19.1.0",
    "react-dom": "^19.1.0"
  },
  "devDependencies": {
    "@types/react": "^19.1.6",
    "@types/react-dom": "^19.1.5"
  }
}
```

## 모노레포에서의 React 19 설정

### 1. UI 컴포넌트 패키지 설정

```json
// packages/ui-components/package.json
{
  "name": "@project/ui-components",
  "peerDependencies": {
    "react": ">=19",
    "react-dom": ">=19"
  },
  "devDependencies": {
    "@types/react": "^19.1.6",
    "@types/react-dom": "^19.1.5",
    "react": "^19.1.0",
    "react-dom": "^19.1.0"
  }
}
```

### 2. 애플리케이션 설정

```json
// apps/frontend/package.json
{
  "name": "@project/frontend",
  "dependencies": {
    "@project/ui-components": "workspace:*",
    "react": "^19.1.0",
    "react-dom": "^19.1.0"
  },
  "devDependencies": {
    "@types/react": "^19.1.6",
    "@types/react-dom": "^19.1.5"
  }
}
```

### 3. 컴포넌트 작성 방법

```typescript
// packages/ui-components/src/components/Button.tsx
import React from 'react';

export interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  variant?: 'primary' | 'secondary';
}

// React 19에서는 React.FC 사용을 권장하지 않음
const Button = ({ children, onClick, variant = 'primary' }: ButtonProps) => {
  return (
    <button 
      className={`btn btn-${variant}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
};

export default Button;
```

## 호환성 문제 해결

### 1. Testing Library 호환성

현재 @testing-library/react는 React 18을 기준으로 하므로 경고가 발생할 수 있습니다:

```bash
✕ unmet peer react@^18.0.0: found 19.1.1
```

**해결방법**:
- 기능상 문제없으므로 경고 무시
- 향후 React 19 지원 버전으로 업데이트

### 2. 써드파티 라이브러리 호환성

React 19와 호환되지 않는 라이브러리가 있을 수 있습니다:

```bash
# 호환성 확인
pnpm ls react
pnpm ls react-dom
```

### 3. ESLint React 설정

```javascript
// config/eslint-config/react.mjs
export default [
  {
    settings: {
      react: {
        version: 'detect'  // 자동으로 React 버전 감지
      }
    },
    rules: {
      'react/jsx-uses-react': 'off',      // React 19에서 불필요
      'react/react-in-jsx-scope': 'off'   // React 19에서 불필요
    }
  }
];
```

## 마이그레이션 체크리스트

### React 18 → 19 마이그레이션

- [ ] React 및 React-DOM 버전 업데이트
- [ ] @types/react, @types/react-dom 버전 업데이트
- [ ] JSX.Element → React.JSX.Element 변경
- [ ] ESLint 규칙 업데이트
- [ ] 타입 검사 실행: `pnpm typecheck`
- [ ] 빌드 테스트: `pnpm build`
- [ ] 개발 서버 테스트: `pnpm dev`

### 새 프로젝트 시작

- [ ] React 19 버전 사용
- [ ] TypeScript 설정에서 jsx: "react-jsx" 사용
- [ ] React.JSX.Element 타입 사용
- [ ] peerDependencies에 React 19 명시

## 성능 최적화

### 1. React 19 새 기능 활용

```typescript
// use Hook 활용 (React 19 신규)
import { use } from 'react';

function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);
  return <div>{user.name}</div>;
}
```

### 2. 컴파일러 최적화

```json
// vite.config.ts
export default defineConfig({
  plugins: [
    react({
      // React 19 최적화 설정
      jsxRuntime: 'automatic'
    })
  ]
});
```

React 19는 많은 개선사항을 제공하지만, 모노레포 환경에서는 각 패키지별로 일관된 설정을 유지하는 것이 중요합니다.