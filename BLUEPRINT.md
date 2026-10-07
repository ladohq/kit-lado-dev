# Blueprint: lado-dev

`lado-dev` develops LADO itself for the long term. The human talks to a supervisor that
designs each change with them and delegates it; an architect reviews designs against the
roadmap, developers work test-first, reviewers check each branch, a checker runs the full
check on it, and the human approves designs and merges. Used by the LADO maintainers in a
session started in the LADO repository.

## 1. Requirements

Restored in 0.10.0 from the kit's README, roles and flows; R8 and R9 were made precise in
the triage of run `improve/lado-dev` (2026-10-06).

- **R1** Every change is designed for the long term: root cause, 2–3 options with their
  cost, held against ROADMAP.md and AGENTS.md. *Source:* `agents/supervisor.md` §2.
- **R2** A feature's design is reviewed against the roadmap and the current architecture
  before any code is written. *Source:* `agents/architect.md`, `flows/feature.yaml`
  `architecture`.
- **R3** The human approves every design of a feature and every merge into main.
  *Source:* `flows/feature.yaml` `design_ok`, `merge_ok`; `flows/fix.yaml` `merge_ok`.
- **R4** Code is written test-first, at the lowest test layer that can catch the bug.
  *Source:* `agents/developer.md` §2.
- **R5** Every branch is reviewed by someone other than its author before it is merged.
  *Source:* `agents/reviewer.md`, `agents/supervisor.md` "Every branch … is reviewed".
- **R6** A small, clearly scoped change with known acceptance criteria goes without a
  design step. *Source:* `flows/fix.yaml`.
- **R7** A bug, friction or debt found outside the task is recorded in BACKLOG.md, never
  fixed silently or lost. *Source:* `skills/lado-checks` "Recording …", every role.
- **R8** Development is fast: each role runs a fixed list of commands, written in its own
  role file: the developer fast tests only (the test files it works on, then `make lint`,
  `make test` and the integration/UI files it changed), the reviewer `make lint` only, and
  the heavy checks (`make check` on the branch merged with main, the live tests the change
  needs) run once, in the `verify` step by the checker, before `merge_ok`, so the human
  approves the merge of a branch whose full check is green; red or a conflict goes back to
  `implement`. The merge step runs no check; if main moved since `verify`, the run goes
  back to `verify`. No path-based test selection. *Source:* the human, triage of
  `improve/lado-dev` (the full check only before the merge) and 2026-10-07 (developer fast
  tests only, reviewer almost none, heavy ones at the end in a separate role).
- **R9** A release needs green CI and green live tests on the release commit; no
  separate full local check (CI runs everything `make check` runs). The human's request to
  release allows one direct commit on main, the version bump; then main is pushed, CI on
  it must be green, the live tests of every provider must be green, and only then is the
  tag pushed (pushing the tag publishes to PyPI). Red CI or red live tests: no tag; the fix
  goes through `fix`, then the release goes on from pushing main with the same version. A
  paid live run with a real model needs the human's yes every time; when the human
  declines it at a release, every other provider's live tests must be green, and the human
  decides whether to release. *Source:* triage of `improve/lado-dev` (question 1) and
  `improve/lado-dev-3` (question 2); `skills/lado-checks`; F3.1, F5.2, F6.1 of the 0.10.0
  report; F12.1, F3.9, F6.2 of the 0.10.1 report; F5.12, F3.16, F6.3 of the 0.10.2 report.
- **R10** UI work follows `docs/design/ui.md` and the mockups the human approved, and is
  reviewed against them. *Source:* `agents/supervisor.md` §2, `agents/developer.md` §2,
  `agents/reviewer.md` §2.
- **R11** Clean-room: LADO borrows ideas, never code. *Source:* `skills/lado-checks`
  "Clean-room".

## 2. Starting point

Archetype "Feature with design gate", with three changes:
- an architect reviews the design before the human's gate (R2): the supervisor designs
  with the human, so an independent design review needs a fresh look;
- a second flow, `fix`, is the "Solo + reviewer" shape for small changes (R6), so they do
  not pay for a design step;
- a `verify` step (role `checker`) between `review` and `merge_ok` runs the slow checks
  once (R8), so neither the developer nor the reviewer runs them and the human's merge
  gate sees a green full check.

Flow skeletons, as LADO runs them (added in 0.10.2; `verify` added in 0.11.0):

```yaml
name: feature
start: design
states:
  design:
    agent: supervisor
    outcomes: {ready: architecture}
  architecture:
    agent: architect
    max_visits: 3
    outcomes: {approved: design_ok, changes: design}
  design_ok:
    gate: approval
    outcomes: {approved: implement, rejected: design}
  implement:
    agent: developer
    outcomes: {done: review}
  review:
    agent: reviewer
    max_visits: 3
    outcomes: {approved: verify, changes: implement}
  verify:
    agent: checker
    outcomes: {green: merge_ok, red: implement, conflict: implement}
  merge_ok:
    gate: approval
    outcomes: {approved: merge, rejected: implement}
  merge:
    agent: supervisor
    outcomes: {merged: done, stale: verify}
  done:
    end: true
```

![feature](blueprint-flows/feature.svg)

```yaml
name: fix
start: implement
states:
  implement:
    agent: developer
    outcomes: {done: review}
  review:
    agent: reviewer
    max_visits: 3
    outcomes: {approved: verify, changes: implement}
  verify:
    agent: checker
    outcomes: {green: merge_ok, red: implement, conflict: implement}
  merge_ok:
    gate: approval
    outcomes: {approved: merge, rejected: implement}
  merge:
    agent: supervisor
    outcomes: {merged: done, stale: verify}
  done:
    end: true
```

![fix](blueprint-flows/fix.svg)

## 3. Traceability

| Element | Kind | Covers | Why it exists / why nothing simpler |
|---|---|---|---|
| `supervisor` | role (lead) | R1, R3, R9 | Designs with the human, delegates, merges. |
| `architect` | role | R2 | Read-only design review; the supervisor cannot review its own design. |
| `developer` | role | R4, R7, R8, R10 | Writes the code; one author per run; runs the fast checks. |
| `reviewer` | role | R5, R8, R10 | Read-only, independent of the developer; runs `make lint` only. |
| `checker` | role | R8 | Runs the slow checks once, read-only for code (a clean merge of main is its only write). The developer cannot: the check must run on the branch merged with the main of that moment, after review; the supervisor could, but its turn would block the human's chat for the length of `make check`. |
| `feature` | flow | R1–R5, R8 | A change that needs a design. |
| `feature.design` | work step | R1 | The design with the human. |
| `feature.architecture` | work step | R2 | Independent design review, `max_visits: 3`. |
| `feature.design_ok` | gate | R3 | What to build is the human's call. |
| `feature.implement` | work step | R4, R7, R8 | Fast checks only. |
| `feature.review` | work step | R5, R8 | `make lint` only, `max_visits: 3`. |
| `feature.verify` | work step | R8 | The only `make check` and live tests, on the branch merged with main; red or conflict goes back to `implement`. |
| `feature.merge_ok` | gate | R3 | Merging into main is hard to undo; the human sees a green full check. |
| `feature.merge` | work step | R7, R8 | BACKLOG.md entries, then `--ff-only`; no check; `stale` goes back to `verify`. |
| `fix` | flow | R5, R6, R8 | The same as `feature` without design and architecture. |
| `fix.implement`, `fix.review`, `fix.verify`, `fix.merge_ok`, `fix.merge` | steps, gate | as in `feature` | |
| `lado-checks` | own skill | R7, R8, R9, R11 | One place for how to run and read checks, merging, releasing, BACKLOG.md and clean-room; the commands each role runs are in that role's file only. |
| `grilling`, `writing-for-agents` | skills (supervisor) | R1, R3 | Rounds of the human's decisions; briefs for agents. |
| `codebase-design` | skill (architect) | R2 | Vocabulary for module boundaries. |
| `tdd`, `diagnosing-bugs`, `verification-before-completion`, `receiving-code-review` | skills (developer) | R4, R5 | Test-first, bug diagnosis, evidence before done, handling review findings. |
| `verification-before-completion` | skill (checker) | R8 | Evidence before reporting `green`. |
| `frontend-design` | skill (developer) | R10 | Quality floor for what the mockups leave open. |
| `critique-*` (6), `feedback-patterns`, `loading-states`, `error-handling-ux`, `navigation-patterns`, `state-machine` | skills (reviewer) | R10 | Reviewing a UI change. |

Dropped in 0.10.0: `brainstorming` (supervisor; commits a spec on main without review, and
§2 already asks for options), `domain-modeling` (architect; writes CONTEXT.md and ADRs in a
read-only role). Packs declare each used skill's own folder, not parent folders.

Reverse check:
- R1: supervisor, `feature.design`, `grilling`, `writing-for-agents`.
- R2: architect, `feature.architecture`, `codebase-design`.
- R3: `feature.design_ok`, `feature.merge_ok`, `fix.merge_ok`.
- R4: developer, `tdd`, `diagnosing-bugs`, `verification-before-completion`.
- R5: reviewer, `feature.review`, `fix.review`, `receiving-code-review`.
- R6: `fix`.
- R7: `lado-checks`, every role, `implement`, `merge`.
- R8: developer, reviewer, checker, `lado-checks`, `implement`, `review`, `verify`, `merge`
  of both flows.
- R9: supervisor §4, `lado-checks`.
- R10: developer, reviewer, `frontend-design`, the UI review skills.
- R11: `lado-checks`.

## 4. Complexity budget

Output of the budget script for 0.11.0.

| Measure | Value | Zone | Reason, when not green |
|---|---|---|---|
| Worker roles (not supervisor) | 4 | yellow | `checker` added in 0.11.0 (R8): the human's decision of 2026-10-07 to run the heavy checks once, at the end, in a separate role. |
| Work steps in `feature` | 6 | yellow | `verify` added in 0.11.0 (R8), same decision. |
| Work steps in `fix` | 4 | green | |
| Gates in `feature` | 2 | green | |
| Gates in `fix` | 1 | green | |
| Words in the longest role prompt | 919 (developer) | yellow | The human asked for each role's exact commands in its own file (2026-10-07); the developer's list (section 5) is the longest. |
| Words in the lead's prompt | 819 | green | |
| Own skills | 1 | green | |
| MCP servers | 0 | green | |
| Similar paragraphs | 3 pairs | yellow | Kept (the human, triage of `improve/lado-dev`): `feature` and `fix` are two flows by design (R2, R6); their `review` and `implement` differ in where the ACs come from (the design note or the task), and each `do` must read on its own. The architect/reviewer openings say the same about runs for two different roles. |

## 5. Change log

| Date | Version | Change | ← Fact (session, run, metric or report) |
|---|---|---|---|
| 2026-10-07 | 0.11.1 | Checker: a red `make check` is no longer rerun whole; only what failed is rerun, up to 2 times: a failed make target (`lint`, `test-js`, `web`, `browser`) by itself, each failed pytest group's `FAILED <node id>` tests with the group's `-m` and `-n0`, a collection error or a group without `FAILED` lines (crash, timeout) as its whole command once; in a kit's own repo `lado kits check .` again. Passes on a rerun → flaky under **Found on the way**, outcome `green`; fails all 3 times → code or test, `red`. `lado-checks` "When a check fails" points to the checker's section 2 for the details. | The human, 2026-10-07, after comparing with the tessera reference: rerun only the failed tests (a whole `make check` rerun costs 3–4 min). |
| 2026-10-07 | 0.11.0 | Test tiers by role: `lado-checks` loses the path table and the import search; each role file lists its exact commands (developer: the test files it works on, then `make lint`, `make test`, the integration/UI files it changed, `make test-js`/`make web` by path, no live, no `make check`; reviewer: `make lint` and the developer's report). New role `checker` and step `verify` between `review` and `merge_ok` in both flows: `git merge main`, `make check` (rerun once when red), the live tests the change needs with the human's yes for a paid run; outcomes `green`/`red`/`conflict`. `merge`: BACKLOG.md entries, `--ff-only`, no check, `stale` → `verify`. `make check` described as three pytest runs one after another, all run, `check: failed: <groups>`. R8 rewritten. | The human's request of 2026-10-07: tests run too often and too slowly; developer fast tests only, reviewer almost none, heavy ones only at the end in a separate role, release as minor 0.11.0; LADO e518427 (`make check` sequential). |
| 2026-10-07 | 0.10.3 | Release of LADO: live tests of every provider (on a no to Claude, every other one); red CI or live after pushing main → no tag, fix through `fix`, push main again with the same version. Developer: a non-live check that cannot run for an environment cause → BLOCKED. Targeted tests: a changed conftest or helper under `tests/` runs its folder's `test_*.py`; a `web/` file that names no screen runs all of `tests/ui/`; every row that matches a path; more than 10 unit hits counted over both searches. Live tests needed for `providers/`, `hooks.py`, `mcp_server.py`, `runtime.py`, `agent_env.py`, `tests/live/`. Merge Done and a BACKLOG.md-only commit agree. R9 updated. | F3.12–F3.18, F5.11–F5.13, F6.3, F9.1, F10.1 of `kit-reports/lado-dev-0.10.2-2026-10-07.md`; the human (triage of `improve/lado-dev-4`, questions 1–2). |
| 2026-10-07 | 0.10.2 | `lado-checks`: one short rule for targeted tests (each `test_*.py` the import search finds, with its folder's `-m`; conftest and helpers pull in nothing; no hit or more than 10 unit files → `make test`; no module lists for a layer); a live provider skipped for an environment cause is a concern for `merge_ok`; on a no to Claude only the providers the change needs; at merge a red `make check` is rerun once before it is classified, and a BACKLOG.md-only commit after a green check needs no new check; the BACKLOG.md template carries `Size:`/`Why here:`. Supervisor: release order (version commit on main, push main, green CI, then the tag); an environment message from a worker leaves its step open; `flow_cancel` finishes workers and keeps worktree and branch. R9 updated. Flow skeletons added to §2. | The human: simplify instead of patching (triage of `improve/lado-dev-3`); F3.3, F3.7, F5.8, F3.8, F3.9, F3.10, F3.11, F12.1, F7.1, F5.9, F5.10, F6.2 of `kit-reports/lado-dev-0.10.1-2026-10-06.md`. |
| 2026-10-06 | 0.10.1 | `lado-checks`: live tests without Claude and per provider, a skipped provider is not green, an exact threshold for targeted tests and what `make test` replaces, no UI/integration suites pulled by a provider change, flaky at merge, a failing check of any kind, `-n auto` not for live, who acts on "When a check fails", BACKLOG.md tiers, the run's branch kept after a cancel. Reviewer: approved mockups win over `critique-*`. Developer: BACKLOG.md items outside a run go to the supervisor. Supervisor: the developer changes `docs/design/ui.md` on the run's branch. R9 updated. | F3.1–F3.6, F5.1–F5.7, F6.1, F2.1 of `kit-reports/lado-dev-0.10.0-2026-10-06.md`. |
| 2026-10-06 | 0.10.0 | Restored this blueprint. | No BLUEPRINT.md; triage of `improve/lado-dev`. |
| 2026-10-06 | 0.10.0 | Checks: targeted during `implement` and `review` by a "changed path → what to run" table in `lado-checks`; full `make check` only, and always, at `merge` (red: code/test → `red`, environment → the human); no separate full check before a release. Roles and flows name no check commands. | The human's request; `make test` takes 3 min 41 s; F3.3, F5.1, F6.1, F6.2, F6.3 of the 0.9.4 report. |
| 2026-10-06 | 0.10.0 | Reviewer: a red check is classified, environment goes to the supervisor; "Yes" when no Critical or Important is open. | F3.1, F3.2 of the 0.9.4 report. |
| 2026-10-06 | 0.10.0 | Dropped `brainstorming` and `domain-modeling`; seams under test named in the design or brief, otherwise the developer picks them by AGENTS.md's layers and lists them. | F5.2, F1.1, F5.3 of the 0.9.4 report. |
| 2026-10-06 | 0.10.0 | Each used skill declared by its own folder. | F10.1 of the 0.9.4 report. |
