# TypeScript 설정 가이드

TypeScript 컴파일러 설정과 프로젝트별 설정 패턴을 다룹니다.

## TypeScript 설정 가이드

### 애플리케이션 모듈 (apps/\*) - React 19 대응

```json
{
  "compilerOptions": {
    "target": "ES2016",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* 번들러 설정 */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",  // React 19 권장 설정

    /* 코드 품질 */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,

    /* 경로 매핑 */
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@assets/*": ["src/assets/*"],
      "@project/ui-components": ["../../packages/ui-components/src"]
    },
    "typeRoots": ["./types", "./node_modules/@types"]
  },
  "include": [
    "vite-env.d.ts",
    "vite.config.ts",
    "src/**/*.js",
    "src/**/*.jsx",
    "src/**/*.ts",
    "src/**/*.tsx",
    "types/**/*.ts"
  ],
  "exclude": ["node_modules", "dist"]
}
```

>  **React 19 주의사항**: 
> - JSX 타입: `React.JSX.Element` 사용
> - 경로 매핑에서 `/*` 제거 (직접 참조)

### 라이브러리 모듈 (packages/\*) - React 19 대응

```json
{
  "compilerOptions": {
    "target": "ES2016",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* 번들러 모드 */
    "moduleResolution": "bundler",
    "preserveSymlinks": true,
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    /* 린팅 */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,

    /* 경로 매핑 */
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },

    /* 선언 파일 생성 (라이브러리용) */
    "declaration": true,
    "declarationMap": true,
    "outDir": "dist"
  },
  "include": ["src/**/*.ts", "src/**/*.tsx", "vite-env.d.ts"],
  "exclude": ["node_modules", "dist", "dev", "**/*.test.ts", "**/*.test.tsx"]
}
```

>  **라이브러리 패키지 주의사항**: 
> - React 타입 의존성 (`@types/react`, `@types/react-dom`) 명시적 설치 필요
> - peerDependencies에 React 19 이상 명시

### tsconfig.build.json (빌드 전용)

```json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src",
    "moduleResolution": "node",
    "declaration": true,
    "declarationMap": true,
    "emitDeclarationOnly": false,
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "dev", "**/*.test.ts", "**/*.test.tsx"]
}
```

## 데모 앱에서 라이브러리 참조

```json
// apps/shared-lib-demo/tsconfig.json
{
  "compilerOptions": {
    "paths": {
      // 개발 중에는 소스 직접 참조 (빠른 피드백)
      "@project/shared-lib": ["../../packages/shared-lib/src/index.ts"]
      // 프로덕션에서는 빌드된 파일 참조
      // "@project/shared-lib": ["../../packages/shared-lib/dist/index.d.ts"]
    }
  }
}
```

## 주의사항

1. **모듈 네이밍**: 모든 모듈은 `@project/` 접두사 사용
2. **TypeScript 설정**: 각 모듈은 독립적인 tsconfig.json 보유
3. **React 19 사용 시**: 
   - JSX 타입: `React.JSX.Element` 사용
   - 필수 의존성: `@types/react`, `@types/react-dom` 명시적 설치
4. **NestJS 사용 시**: 
   - 엔터티/DTO 클래스에서 `!` (Definite Assignment Assertion) 사용
   - strict 모드에서 프로퍼티 초기화 오류 방지
5. **경로 매핑**: 
   - include 경로를 각 패키지에서 명시적으로 재정의
   - 상대 경로 문제 방지
