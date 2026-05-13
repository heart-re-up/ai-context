# Convention · component

[커스텀 훅](./convention-hook.md)과 짝을 이룬다. **이름·props 타입·구조 분해·불리언**은 [계약·네이밍](./convention-contract.md)을 따른다.

**OwnProps + `ComponentProps*`(호스트 props) 타입·분할·루트 전달**은 한 주제로 [Own과 호스트 props](./convention-own-host-props.md)에 모아 두었다.

# React component principles

- **역할**: 렌더·이벤트 바인딩·가벼운 파생값은 컴포넌트에 두고, 구독·복잡한 동기화·DOM 생명주기 로직은 **커스텀 훅**으로 옮긴다. “두꺼운” 컴포넌트보다 훅 + 얇은 UI를 선호한다.
- **ref**: 부모가 DOM에 닿아야 할 때는 React 19에서는 가능한 한 **`ref`를 일반 prop**으로 받아 내부 DOM/포워딩 대상에 연결한다. (`forwardRef`는 레거시·호환용으로만 제한적으로 사용.) “DOM을 어떻게 구독할지”는 훅이 **`RefCallback`을 반환**하는 규칙과 짝을 이루고, 컴포넌트는 그 콜백을 `ref={...}`에 넘기는 쪽에 가깝다. (상세는 [커스텀 훅 규약](./convention-hook.md)의 **ref (수신이 아닌 반환)**.)
- **성능**: `useMemo`/`useCallback`으로 JSX 전체를 감싸기보다, **자식에게 넘기는 객체·함수**나 **비싼 계산**만 필요할 때 안정화한다. `key`로 리마운트를 제어하는 것이 더 맞는 경우 메모이제이션으로 숨기지 않는다. (객체·필드 메모 원칙은 [계약](./convention-contract.md)의 반환 절을 참고.)
- **조건부 훅**: `use*` 호출 순서가 바뀌지 않게 유지한다. 조건 분기 안에서 훅을 새로 호출하지 않는다.
