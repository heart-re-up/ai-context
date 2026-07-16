# Convention · hook

UI 쪽 규약은 [컴포넌트](./convention-component.md)와 같이 본다.

**입출력 타입·네이밍·구조 분해·불리언·반환 객체**는 [계약·네이밍](./convention-contract.md)을 따른다.

# React custom hook principles

## 예시

```ts
// 네이밍: useXxx 훅 → UseXxxProps / UseXxxReturns. DOM·인스턴스는 T extends Element 로 ref·콜백 시그니처를 맞춘다.
// 입출력: 인자는 Props 객체 하나, 반환은 Returns 객체 하나(둘 다 명시 타입).
// ref: 인스턴스는 Props의 RefObject로 받지 않는다. 훅이 RefCallback을 반환하고 소비 측이 ref={...}에 걸면 연결·해제 시점(획득·null)을 본다. (아래 ## ref)

export type UseCustomHookProps<T extends Element> = {
  enabled?: boolean;
  value?: string;
  onChange?: (value: string) => void;
  onMount?: () => void;
  onUnmount?: () => void;
  // 부모 알림용(선택). ref 수신과 무관—연결 자체는 아래 반환 RefCallback이 담당.
  onElementChange?: (el: T | null) => void;
};

// 인스턴스 연결용은 RefCallback 권장(RefObject 수신 금지 가이드와 동일).
export type UseCustomHookReturns<T extends Element> = {
  setCustomRef: RefCallback<T>;
};

export const useCustomHook = <T extends Element>(
  props: UseCustomHookProps<T>,
): UseCustomHookReturns<T> => {
  // 계약은 매개변수 목록이 아니라 *Props 타입 한곳에만 둔다. 본문 첫줄에서 props만 구조 분해한다.
  const {
    // 불리언: 기본이 “꺼짐”이 자연스럽게 enabled=false. 기본이 “켜짐”이 자연스러우면 disabled·preventClose처럼 부정형으로 뒤집는다.
    enabled = false,
    // Prop 접미사: 내부 이름과 겹치거나 “외부에서 온 값”을 분리할 때만 쓴다(남발하지 않음).
    value: valueProp,
    onElementChange: onElementChangeProp,
    onChange,
    onMount,
    onUnmount,
  } = props;

  // props를 그대로 쓰지 않고 내부에서 합성·파생할 때는 다른 이름(예: valueProp → value).
  const value = valueProp ?? "default value";
  const [element, setElement] = useState<T | null>(null);

  // 부모 콜백 최신본은 쓰되, 매 렌더마다 바뀌는 참조로 effect가 도는 것을 막기 위해 이벤트 경계(useEffectEvent 등).
  const onElementChange = useEffectEvent((el: T) => onElementChangeProp?.(el));

  // 반환하는 RefCallback은 useCallback으로 안정화 → 소비 측 deps에 넣어도 참조가 불필요하게 바뀌지 않게.
  // ref 시점에 setState와 부모 알림을 한 블록에 둬 DOM 연결을 일관되게 유지한다.
  const setCustomRef = useCallback<RefCallback<T>>(
    (instance: T | null) => {
      setElement(instance);
      onElementChange(instance);
    },
    [setElement, onElementChange],
  );

  // 레이아웃·DOM에 맞춘 동기 알림이 필요하면 useLayoutEffect + element 등 실제 변화와 묶는다.
  useLayoutEffect(() => {
    onElementChange(element);
  }, [element, onElementChange]);

  useLayoutEffect(() => {
    if (!enabled) {
      return; // 꺼져 있으면 구독·마운트 로직 자체를 타지 않음(빠른 반환).
    }
    onMount?.();
    return () => {
      onUnmount?.(); // effect 재실행·언마운트 시 이전 effect의 정리만 실행되도록 cleanup으로 표현.
    };
  }, [enabled, onMount]);

  // 반환 객체 전체는 useMemo하지 않는다. 객체 안의 함수·참조만 필요한 만큼 useCallback 등으로 안정화.
  return {
    setCustomRef,
  };
};
```

## ref (수신이 아닌 반환)

- DOM·외부 라이브러리 인스턴스를 훅에 넘길 때 **`RefObject`를 props로 받지 않는다.** “부모 ref에 훅이 `useEffect(() => ref.current)`만 붙는다” 식은 피한다.
- `RefObject`의 `.current`만 바뀌는 경우 **렌더가 일어나지 않아** 훅 본문·`useEffect`가 그 순간에 다시 실행되지 않는다. 외부에서 ref를 획득해 넘겨도, 인스턴스가 붙는 타이밍에 훅 로직이 자동으로 돌지 않을 수 있다.
- 그래서 훅은 **`RefCallback`을 반환**해 소비 측이 `ref={setXxxRef}`처럼 연결하도록 한다. React가 마운트·업데이트·언마운트 시 콜백을 호출하므로 **인스턴스 획득과 유실(`null`)**을 훅 안에서 확실히 알 수 있다.
- 그 콜백에서 받은 인스턴스를 내부 `useRef`에만 보관할지, `useState`로 올려 effect·레이아웃과 묶을지는 **훅 구현이 자유롭게** 정한다.
- 부모에게만 알리면 되는 경우 `onElementChange` 같은 **옵션 콜백 prop**은 ref 수신과 별개로 둘 수 있다. 연결 책임은 여전히 반환 `RefCallback`에 둔다.

## 콜백 안정성과 반환 RefCallback

- 부모가 넘긴 콜백을 effect나 ref 콜백 안에서 **최신 버전**을 쓰되, 그 콜백이 매 렌더마다 바뀌어 effect가 불필요하게 재실행되지 않게 하려면 `useEffectEvent` 같은 패턴으로 “이벤트 핸들러” 경계를 둔다. 내부에서만 최신 ref를 읽는 용도로 쓰고, effect의 의존성에는 넣지 않는 방향이 일반적이다.
- 반환하는 `RefCallback`은 `useCallback`으로 감싸 외부가 `deps`에 넣어도 참조가 불필요하게 바뀌지 않게 한다. 콜백 본문에서는 상태 갱신과(필요 시) 부모 알림을 한곳에 모아 **연결 시점**을 일관되게 유지한다.

## effect 선택

- DOM 측정·동기 레이아웃과 맞춰야 하는 알림은 `useLayoutEffect`를 고려한다. 마운트 직후 한 번 맞춰야 하는 콜백이면 예시처럼 `element` 변경과 묶는다.
- `enabled`처럼 기능 켜짐/꺼짐에 따른 구독·정리는 effect 안에서 먼저 비활성 분기로 빠르게 반환하고, 활성일 때만 `onMount`·정리에서 `onUnmount`를 호출한다. 정리 함수는 항상 effect가 반환하는 함수로만 표현해 재실행 시 이전 구독이 남지 않게 한다.

## 요약

- 공통: [계약·네이밍](./convention-contract.md)(입출력, 네이밍, 구조 분해, 불리언, 반환 객체).
- **ref**: `RefObject`를 props로 받지 않고 `RefCallback`을 반환해 연결·해제 시점을 확보한다. 내부 저장은 `useRef`/`useState` 등 훅이 정한다.
- **구현**: effect는 빠른 반환과 정리 함수로 자원 생명주기를 명확히 → 반환 `RefCallback`·부모 콜백은 `useCallback`과 `useEffectEvent` 등으로 의존성 지옥을 줄인다.
