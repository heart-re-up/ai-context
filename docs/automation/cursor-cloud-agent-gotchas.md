# Cursor Cloud Agent — 자주 겪는 함정

Cursor Cloud Agent(Automations, Cloud VM)를 GitHub Issue 트리거·원격 개발 환경으로 쓸 때 겪는 비자명한 문제와 해결 순서.

## 1. GitHub Issue 트리거 설정

### 최종 동작하는 설정 (요약)

| 항목 | 값 |
|---|---|
| 트리거 | GitHub → **Issue comment** (Label 트리거 아님) |
| 매칭 문구 | `!cursor` (regex) — `@cursor`는 절대 쓰지 않음 |
| Agent Instructions | 트리거 payload에 이슈 본문이 없다고 가정하고 `gh issue view`로 직접 조회하도록 명시 |
| GitHub 인증 | Cloud Agent 기본 토큰 대신 **Classic PAT를 `GH_TOKEN` 환경변수(My Secrets)로 주입** |

### 문제 1 — Label 트리거는 PR에만 걸리고 Issue에는 안 걸림

Automations 편집기에서 라벨 트리거를 추가하면 기본으로 `Label Added on PRs in <repo>` 형태로 잡힌다. 이 드롭다운에는 **Issues로 바꾸는 옵션이 없다** — GitHub PR과 non-PR Issue의 라벨 트리거는 UI상 별개 취급이고, 이 편집기 화면에서는 PR 쪽만 노출된다.

**해결**: 라벨 트리거를 포기하고 **Issue comment** 트리거를 쓴다 (이슈에 코멘트가 달릴 때 발동).

### 문제 2 — `@cursor`는 공식 온디맨드 에이전트 멘션과 충돌

Issue comment 트리거의 매칭 문구로 `@cursor`를 쓰면, Cursor 공식 GitHub 앱이 이미 이 멘션을 "온디맨드 에이전트 호출"로 예약해두고 있어서 커스텀 Automation이 아니라 공식 기능이 반응한다.

**해결**: `@`로 시작하지 않는 별도 문구를 쓴다. 예: `!cursor`. Matching 필드에 정규식으로 등록.

### 문제 3 — 에이전트가 트리거된 이슈의 본문을 못 읽음

Cursor Automations는 `{{issue.body}}` 같은 템플릿 변수를 지원하지 않는다. 트리거 payload에 이벤트 메타데이터(코멘트 내용, 이슈 번호 등)는 들어가지만, **이슈 원본 본문까지 자동으로 포함된다는 보장이 없다**.

**해결**: Agent Instructions에 아래처럼 명시해서, 본문이 이미 주어졌다고 가정하지 않고 직접 조회하게 한다.

```
이 Automation을 트리거한 이슈 번호를 트리거 payload에서 확인해라.
이슈 본문이 이미 컨텍스트에 있다고 가정하지 말고,
`gh issue view <이슈번호> --json title,body,labels,comments`로
이슈 제목·본문·코멘트를 직접 조회해서 요구사항을 파악해라.
```

### 문제 4 — Cloud Agent 기본 토큰에 Issues/PR 권한이 없음

Cursor GitHub App의 설치 권한에는 `Issues: Read and write`, `Pull requests: Read and write`가 이미 포함돼 있지만, **Cloud Agent 샌드박스에 실제로 발급되는 런타임 토큰은 그보다 좁게 스코프된 상태**라 `gh issue view`, `gh pr comment` 등이 `403 Resource not accessible by integration`으로 실패한다. Cursor 팀이 공식 포럼에서 인지하고 있는 버그다.

**해결 (공식 워크어라운드)**: GitHub Personal Access Token을 따로 발급해서 `GH_TOKEN` 환경변수로 주입하면, 에이전트 샌드박스의 `gh` CLI가 내장 토큰 대신 이 토큰을 우선 사용한다.

1. Classic PAT 발급 — `repo` 스코프
   - Fine-grained PAT도 되지만, **Resource owner가 기본으로 개인 계정으로 잡혀서** 조직 리포가 Repository access 목록에 안 보이는 문제가 있을 수 있다. Classic PAT는 이 개념이 없어 더 간단하다.
   - 필요 권한: `Contents`(RW), `Issues`(RW), `Pull requests`(RW), `Metadata`(R, 자동 포함). `.github/workflows/*` 수정이 필요한 작업이면 `Workflows`(RW)도 추가.
2. Cursor 대시보드 → **Settings → Cloud Agents → My Secrets** → `GH_TOKEN` 이름으로 등록, Repositories는 필요한 범위로.
3. Automation의 **Permission scope**가 `Private`/`Team Visible`이면 본인 계정의 My Secrets가 그대로 적용된다. `Team Owned`로 승격하면 팀 공유 서비스 계정 identity로 실행되므로 팀 시크릿 쪽에 별도로 등록해야 한다.

**확인 방법**: 트리거된 에이전트 안에서 `gh auth status`를 실행하면 `Logged in to github.com account <내계정> (GH_TOKEN)`이 active로 뜨는지 확인. `cursor` 계정(`ghs_...` 토큰)이 active면 아직 내장 토큰을 쓰는 중이라는 뜻.

## 2. Cloud VM — Node 버전

Cloud Agent VM의 기본 `node`는 최신 프레임워크(예: React Router v7)가 요구하는 버전보다 낮을 수 있다. dev 서버를 띄우기 전에 프로젝트가 요구하는 Node 버전을 PATH 앞에 둔다:

```bash
# nvm 사용 예
export PATH="$(ls -d ~/.nvm/versions/node/v22.22.*/bin | tail -1):$PATH"
pnpm dev
```

Volta를 쓰는 프로젝트라면 `package.json`의 `volta` 필드가 Cloud VM에서도 적용되는지 확인한다. Turbo TUI 대신 로그를 파일로 보려면 `TURBO_UI=stream pnpm dev`.

## 관련 문서

- [Cursor Automations 공식 문서](https://cursor.com/docs/cloud-agent/automations) — 트리거·Tools·Permission scope 레퍼런스
- [버전 관리](../node/version-management.md) — `.nvmrc`, Volta 등 Node 버전 고정
