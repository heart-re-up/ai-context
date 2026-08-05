# 코드 주석 규약

함수·분기 주석의 기준과 예시. Cursor 규칙: [`comments.mdc`](../../rules/comments.mdc).

## 원칙

- **What이 아니라 Why/역할**: 이미 코드가 말하는 내용을 반복하지 않는다.
- **짧게**: 한두 문장. 긴 배경은 이슈·설계 문서로 보내고, 주석에는 읽기 위치에서 필요한 계약만 남긴다.
- **언어 관례**: TypeScript/JavaScript는 JSDoc(`/** */`), 그 외는 해당 언어의 문서 주석 형식을 쓴다.

## 함수 주석

### 달 것

- 공개 API (export된 함수·메서드·타입에 붙는 동작 설명)
- 분기·재시도·버퍼링·권한·타임아웃처럼 **부수효과나 불변식**이 있는 함수
- 이름만으로는 범위가 안 보이는 헬퍼 (`apply`, `handle`, `process` 등)

### 생략해도 되는 것

- 본문이 한 줄이고 이름이 역할을 말하는 래퍼
- 단순 필드 getter/setter, 자명한 매핑

```ts
// ❌ BAD — 코드를 번역함
/** x에 1을 더한다 */
function increment(x: number) {
  return x + 1;
}

// ✅ GOOD — 역할·계약
/** 재연결 대기열에 넣고, 연결이 살아 있으면 즉시 flush한다. */
function enqueueOrFlush(frame: Frame) {
  // ...
}

// ✅ OK — 자명 래퍼, 주석 생략
function toIso(date: Date) {
  return date.toISOString();
}
```

## 조건·분기 주석

동작이 갈리는 `if` / `switch` / early-return / 삼항에는 **왜 그 갈래인지**를 남긴다.

```ts
// ❌ BAD — 조건식 재서술
if (user == null) {
  // user가 null이면
  return;
}

// ✅ GOOD — 사유·기대 동작
// 익명 시청자는 presence에 넣지 않는다. 재접속 시 동일 익명 키로 중복 집계된다.
if (user == null) {
  return;
}

// ✅ GOOD — 대안을 고른 이유
// preview는 샘플 이벤트로 idle을 채운다. live는 빈 슬롯을 유지해 실제 수신만 그린다.
if (mode === "preview") {
  return sampleEvents;
}
return [];
```

## 관련 문서

| 문서 | 내용 |
|------|------|
| [convention-contract.md](./convention-contract.md) | 컴포넌트·훅 계약·네이밍 |
| [quality/README.md](../quality/README.md) | ESLint·Prettier 등 품질 도구 |
