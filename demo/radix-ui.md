# Radix UI Themes를 활용한 빠른 데모 구성

React Hooks 라이브러리 데모 구성 시 Radix UI Themes를 활용하여 빠르게 UI를 구성하는 방법입니다.

## 패키지 설치

```json
{
  "dependencies": {
    "@radix-ui/themes": "^3.2.1",
    "@radix-ui/react-accordion": "^1.2.2",
    "@radix-ui/react-separator": "^1.1.1",
    "@radix-ui/react-switch": "^1.1.2",
    "@radix-ui/react-tabs": "^1.1.2"
  }
}
```

## 주요 컴포넌트 사용법

### Card 컴포넌트

❌ **잘못된 사용법** (`@radix-ui/react-card`는 존재하지 않음):

```tsx
import * as Card from "@radix-ui/react-card";

<Card.Root>
  <Card.Header>
    <Card.Title>제목</Card.Title>
  </Card.Header>
  <Card.Content>내용</Card.Content>
</Card.Root>;
```

✅ **올바른 사용법** (`@radix-ui/themes` 사용):

```tsx
import { Card } from "@radix-ui/themes";

<Card className="p-6">
  <div className="pb-4">
    <h3 className="text-lg font-medium">제목</h3>
    <p className="text-sm text-muted-foreground">설명</p>
  </div>
  <div className="space-y-4">{/* 내용 */}</div>
</Card>;
```

### Switch 컴포넌트

```tsx
import * as Switch from "@radix-ui/react-switch";

<Switch.Root
  checked={isEnabled}
  onCheckedChange={setEnabled}
  className="w-11 h-6 bg-gray-200 rounded-full relative data-[state=checked]:bg-primary outline-none cursor-pointer"
>
  <Switch.Thumb className="block w-5 h-5 bg-white rounded-full transition-transform duration-100 translate-x-0.5 will-change-transform data-[state=checked]:translate-x-[22px]" />
</Switch.Root>;
```

### Tabs 컴포넌트

```tsx
import * as Tabs from "@radix-ui/react-tabs";

<Tabs.Root defaultValue="localStorage" className="w-full">
  <Tabs.List className="grid w-full grid-cols-3 bg-muted rounded-lg p-1">
    <Tabs.Trigger value="localStorage">useLocalStorage</Tabs.Trigger>
    <Tabs.Trigger value="debounce">useDebounce</Tabs.Trigger>
    <Tabs.Trigger value="toggle">useToggle</Tabs.Trigger>
  </Tabs.List>

  <Tabs.Content value="localStorage">{/* 내용 */}</Tabs.Content>
</Tabs.Root>;
```

### Accordion 컴포넌트

```tsx
import * as Accordion from "@radix-ui/react-accordion";

<Accordion.Root type="single" collapsible className="w-full">
  <Accordion.Item value="advanced" className="border-b">
    <Accordion.Trigger className="flex flex-1 items-center justify-between py-4 text-sm font-medium transition-all hover:underline [&[data-state=open]>svg]:rotate-180">
      고급 설정
      <svg className="h-4 w-4 shrink-0 text-muted-foreground transition-transform duration-200">
        <path d="m6 9 6 6 6-6" />
      </svg>
    </Accordion.Trigger>
    <Accordion.Content className="overflow-hidden text-sm data-[state=closed]:animate-accordion-up data-[state=open]:animate-accordion-down">
      {/* 내용 */}
    </Accordion.Content>
  </Accordion.Item>
</Accordion.Root>;
```

## Vite 설정에서 청킹

```typescript
// vite.config.ts
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ["react", "react-dom"],
          radix: [
            "@radix-ui/react-accordion",
            "@radix-ui/react-separator",
            "@radix-ui/react-switch",
            "@radix-ui/react-tabs",
            "@radix-ui/themes",
          ],
        },
      },
    },
  },
});
```

## 스타일링 팁

- Radix UI Themes는 CSS 변수 기반으로 작동
- Tailwind CSS와 함께 사용할 때 CSS 변수 활용:

  ```css
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --muted: 210 40% 96%;
    --primary: 222.2 47.4% 11.2%;
  }
  ```

- 컴포넌트별 className 조합으로 스타일 커스터마이징
- `data-[state=*]` 속성을 활용한 상태별 스타일링

이렇게 구성하면 빠르게 전문적인 UI를 가진 데모를 만들 수 있습니다.
