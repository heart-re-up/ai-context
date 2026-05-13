# Convention · contract

컴포넌트와 커스텀 훅이 **같이 따르는** 타입·네이밍·입출력·구조 분해 규약이다. (래퍼의 **OwnProps + `ComponentProps*`**는 [Own과 호스트 props](./convention-own-host-props.md), 컴포넌트 나머지는 [컴포넌트](./convention-component.md), ref·effect·훅 예시는 [훅](./convention-hook.md).)

## 입출력 타입 계약

- **공개 API는 타입 한곳에만 모은다.** 매개변수를 길게 나열하지 않고, `*Props`(및 훅이면 `*Returns`) 타입에 계약을 적는다.
- **컴포넌트**: 인자는 `(props: XProps)` 형태로 **props 객체 하나**를 받고, `type XProps`로 정의한다.
- **훅**: 인자는 **`Props` 객체 하나**, 반환은 **`Returns` 객체 하나**이며 둘 다 명시적 타입(`UseXxxProps`, `UseXxxReturns`)을 쓴다.

## 네이밍 (컴포넌트 vs 훅)

| 구분 | 심볼 | 입력 타입 | 출력 / 반환 |
|------|------|-----------|----------------|
| 컴포넌트 | `Button` 등 **PascalCase** | `ButtonProps` 등 `type {Name}Props` | JSX (ReactElement) |
| 훅 | `useThing` — `use` + PascalCase 도메인 | `UseThingProps` (`use` 제거 후 `Use` 접두 + `Props`) | `UseThingReturns` (`Use` + `Returns`) |

- export 스타일(named / default)은 프로젝트 룰에 따른다.
- DOM 노드·외부 인스턴스를 다룰 때는 `Element` 등 상한을 두고 제네릭 `T`로 맞춘다(훅의 ref·콜백 시그니처와 한 줄에서 일치).

## 구조 분해와 Prop 접미사

- 본문 첫머리에서 **`props`를 구조 분해**한다(훅·컴포넌트 공통).
- props의 값은 그대로 쓰고, 내부에서 합성·파생하는 값은 **다른 이름**을 붙인다(예: `valueProp` → 내부 `value`).
- props 이름과 내부 이름이 겹치면 구조 분해에서 `원래이름: 내부이름`으로만 정리한다.
- **`SomethingProp` 접미사**는 내부와 충돌하거나 “외부에서 온 값”임을 분명히 할 때만 쓴다(남발하지 않는다).

## 불리언과 기본값

- “꺼져 있을 때가 자연스러운” 플래그는 **`false`가 기본**이 되도록 이름을 짓는다. 예: `enabled`, 기본값 `false`. 켜야만 동작하는 효과는 `if (!enabled) return`으로 빠르게 빠져나간다.
- 기본이 켜짐이 자연스러우면 **부정형**으로 뒤집어 기본을 `false`로 맞춘다. 예: `disabled`, `preventClose`. 의존성 배열·조건식이 “기본이 안전한 쪽”으로 수렴하게 한다.

## 반환 객체 (주로 훅)

- 훅 반환은 **단일 객체**이며, 객체 전체를 `useMemo`로 감쌀 필요는 없다.
- 객체 안의 **함수·참조**만 `useCallback`/`useMemo` 등으로 필요한 만큼 안정화한다. 소비 측이 `const { a } = useHook()`처럼 구조 분해해도, 안정화된 필드만 deps에 넣으면 된다.
- 컴포넌트가 “설정 객체” 등을 만들어 자식에게 넘길 때도 **불필요한 전체 메모**보다 **넘기는 필드 단위** 안정화를 우선한다.
