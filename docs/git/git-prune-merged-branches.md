# 병합된 브랜치 정리 (`git prune-*`)

여러 저장소에서 쓰는 개인용 git 확장 명령이다. 특정 모노레포에 종속되지 않는다.

런타임 설치본은 보통 아래 경로에 둔다.

```text
%USERPROFILE%\.local\bin   (Windows)
$HOME/.local/bin           (Git Bash / macOS / Linux)
```

Git Bash는 기본으로 `$HOME/bin`도 PATH에 넣는다. `~.local\bin`이 셸에 안 잡히면 `$HOME/bin`에 같은 파일을 두어도 된다.

---

## 공식 확장 모델

Git은 PATH에서 `git-<이름>` 실행 파일을 찾아 `git <이름>`으로 실행한다. 전역 alias는 필요 없다.

| 파일 | 호출 |
| --- | --- |
| `git-prune-merged-remote` | `git prune-merged-remote` |
| `git-prune-orphan-local` | `git prune-orphan-local` |
| `git-prune-branches` | `git prune-branches` (둘을 순서대로) |
| `git-prune-lib.sh` | 호출하지 않는다. 위 스크립트가 `source` 한다 |

---

## 사용

```bash
git prune-branches              # dry-run (기본)
git prune-branches --apply      # 실제 삭제
git prune-merged-remote --apply
git prune-orphan-local --apply
git prune-orphan-local --all-orphans --apply   # 미병합 orphan까지 (위험)
```

공통 옵션: `--remote origin`, `--base develop`, `--no-fetch`, `--apply`.

---

## 동작 계약

1. **기본은 dry-run.** `--apply` 없이 remote/local을 지우지 않는다.
2. **remote 삭제 대상**은 `origin/develop`(또는 `--base`)에 **이미 병합된** 브랜치만이다. 미병합은 `KEEP`으로만 출력한다.
3. **로컬 orphan 기본 삭제**는 remote에 같은 이름이 없고 **base에 병합된** 것만이다. 푸시하지 않은 미병합 WIP는 지우지 않는다.
4. **보호 브랜치:** 현재 checkout, `main`, `master`, `develop`(또는 `--base`).
5. **fetch 후 로컬 base sync**
   - 현재 브랜치가 base면 `git pull --ff-only`
   - 아니면 checkout/rebase 없이 `git branch -f <base> <remote>/<base>` (fast-forward만)
   - local ahead/diverged면 강제하지 않는다
6. **squash merge**만 된 브랜치는 `--merged`/ancestor 판정에 안 잡힌다.
7. `git prune-branches` 순서: merged-remote(fetch+sync) → orphan-local(`--no-fetch`).

---

## Windows / Cursor PATH

- 사용자 PATH에 `C:\Users\<user>\.local\bin`을 추가한다. `%USERPROFILE%` 문자열을 PATH에 그대로 넣지 말고, 펼쳐진 경로를 쓴다. Git Bash는 `%USERPROFILE%`를 해석하지 않는다.
- Cursor(또는 터미널 앱)는 **프로세스 시작 시점**의 PATH를 물려받는다. PATH를 바꾼 뒤에는 Cursor를 완전히 종료했다가 다시 켜야 통합 터미널에 반영된다. 탭만 새로 여는 것으로는 부족할 수 있다.
- 확인:

```bash
echo "$PATH" | tr ':' '\n' | grep '\.local/bin'
git prune-branches
```

---

## 파일 역할 (설치본)

| 파일 | 역할 |
| --- | --- |
| `git-prune-merged-remote` | 병합된 remote 브랜치 목록·삭제 |
| `git-prune-orphan-local` | remote 없는 로컬 orphan 목록·삭제 |
| `git-prune-branches` | 위 둘을 순서대로 호출 |
| `git-prune-lib.sh` | 로컬 base fast-forward 헬퍼 |

설치·갱신은 이 디렉터리의 파일을 `~/.local/bin`(또는 `$HOME/bin`)에 복사하고 실행 권한을 주면 된다. 재작성할 때는 위 동작 계약을 유지한다.

---

## 갱신 이력

- 2026-08-12: 개인 `git-*` PATH 확장으로 정리. 저장소 스크립트 대신 이 문서로 관리한다.
