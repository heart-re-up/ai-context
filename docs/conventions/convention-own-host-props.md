# Convention · own and host props

래퍼 컴포넌트가 **자체 API(OwnProps)**와 **내장 요소가 그대로 받을 수 있는 속성(호스트 props, `ComponentPropsWithRef` / `ComponentPropsWithoutRef` 등)**을 **한 props 객체**로 노출할 때의 타입·분할·루트 전달 규약이다. (역할·ref·성능 등은 [컴포넌트](./convention-component.md), 공통 계약은 [contract](./convention-contract.md).)

## 주제 정의

- **OwnProps**: 이 컴포넌트만의 의미—`title`, `roundness`, `variant` 등 DOM에 원래 없거나 의미가 다른 필드.
- **호스트 props**: 루트 DOM(또는 `as`로 고른 intrinsic)이 그대로 받는 속성—`className`, `style`, `ref`, `data-*`, ARIA, `onClick` 등. 타입은 보통 **`ComponentPropsWithRef<"div">`** 또는 **`ComponentPropsWithoutRef<"button">`**처럼 **태그 리터럴**로 고른다.
- **합친 공개 타입**: `type XProps = ComponentPropsWithRef<"div"> & XOwnProps` — 소비자는 Own과 호스트를 **한 객체**로 넘긴다.

## 단일 루트 규칙

- **`...rootProps`(rest)**는 **호스트 계약의 기준이 되는 루트 요소 하나**에만 펼친다. `className`·`style`·`ref`·`data-*`·ARIA·DOM 이벤트는 전부 이 루트로 전달한다.
- 안쪽 래퍼에까지 전부 펼치면 `id`·`ref` 등이 의도와 다르게 갈 수 있다. 안쪽에 일부만 넘겨야 하면 **명시적으로** 나눈다.
- `children`은 Own에 넣지 않아도 되지만, **루트 바로 아래에 두지 않을 레이아웃**이면 구조 분해로 빼고 나머지 호스트만 루트에 붙인다.

## 타입 조합과 이름 충돌

- **기본**: `type XOwnProps = { ... }; type XProps = ComponentPropsWithRef<"div"> & XOwnProps`.
- **Own 키가 HTML 속성과 겹칠 때** (`title` 등): 교차 타입만으로는 의미가 모호해질 수 있다. **`Omit<ComponentPropsWithRef<"div">, keyof XOwnProps> & XOwnProps`**처럼 호스트 쪽에서 Own 키를 빼고 Own을 다시 붙여 **Own 의미가 우선**이게 하거나, Own 이름을 `heading`처럼 **호스트와 안 겹치게** 짓는다.
- **폴리모픽(태그만 바꿀 때)**: `as`로 `div`/`section` 등을 바꾸면 제네릭에 `ElementType` 등을 두고 **`ComponentPropsWithRef<T>`**로 호스트 쪽을 맞춘다. (프로젝트 공통 추상화가 있으면 그걸 따른다.)

## 왜 나누는가

- 소비자는 “카드 제목” 같은 **도메인 props**과 “이 div에 줄 `onClick`” 같은 **호스트 props**를 한 번에 넘기고, 구현은 **루트 태그 규약**을 깨지 않게 유지한다.

## 분할 방법

- **적은 Own**: `const { a, b, children, ...rootProps } = props` 한 번 → `<div {...rootProps}>`.
- **금지**: `const { p1, …, …rest } = props` 뒤에 **`const ownProps = { p1, … }`로 같은 키를 두 번 객체화**하지 않는다. `pick(props, ownKeys)` 등으로 한 번에 만든다.
- **Own이 많을 때**: `ownKeys`를 `as const satisfies readonly (keyof XOwnProps)[]`로 두고, **`lodash-es`의 `pick`/`omit`**으로 `ownProps`와 `rootProps`를 만든다. 타입과 런타임 키 목록이 어긋나면 컴파일에서 잡히게 한다. `children`처럼 Own 타입에 없지만 루트에 바로 두지 않을 키는 `ownKeys`에 넣거나, `pick`/`omit` 뒤 한 번 더 구조 분해한다.
- **“유효한 DOM 키만 남기기”가 목적**이면 `@emotion/is-prop-valid` 등 **호스트 유효성 필터**를 쓸 수 있다. **Own과 호스트 경계를 나누는 것**이 목적이면 **Own 키 목록·`Omit<…, keyof XOwnProps>`**를 우선한다.

## 컴포지션·API 모양

- 평탄한 Own이 **한 컴포넌트로 읽기 어려울 만큼 커지면**(개수만이 아니라 응집도가 깨질 때) 필드 그룹 컴포넌트·슬롯·전용 훅으로 쪼갠다.
- **`appearance`·`fieldOptions`**처럼 의미 단위 객체로 묶거나, 호스트만 **`rootProps`/`htmlProps` 한 객체**로 받는 API도 고려한다.

## 예시

```tsx
// import type { ComponentPropsWithRef } from "react";

export type CardOwnProps = {
  title?: string;
  roundness?: number;
  elevation?: number;
};

export type CardProps = ComponentPropsWithRef<"div"> & CardOwnProps;

export const Card = (props: CardProps) => {
  const { title, roundness = 12, elevation = 4, children, ...divProps } = props;
  return (
    <div {...divProps}>
      {title != null && title !== "" && <div>{title}</div>}
      <div>{children}</div>
    </div>
  );
};
```

```tsx
// import type { ComponentPropsWithRef } from "react";
// import { omit, pick } from "lodash-es";
// CardOwnProps, CardProps 는 위 블록과 동일.

const cardOwnKeys = [
  "title",
  "roundness",
  "elevation",
] as const satisfies ReadonlyArray<keyof CardOwnProps>;

export const CardSplit = (props: CardProps) => {
  const ownProps = pick(props, [...cardOwnKeys]);
  const rootProps = omit(props, [...cardOwnKeys]);
  const { title, roundness = 12, elevation = 4 } = ownProps;
  const { children, ...divProps } = rootProps;
  return (
    <div {...divProps}>
      {title != null && title !== "" && <div>{title}</div>}
      <div>{children}</div>
    </div>
  );
};
```
