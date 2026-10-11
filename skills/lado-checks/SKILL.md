---
name: lado-checks
description: How LADO checks are run (which pytest marker each test folder needs, `make check`, never piping a gating check), how the merge step merges a run's branch that `verify` already checked, how a release is checked, how to read a failure, and how to record a bug, friction or debt in BACKLOG.md and who records it. Each role's own file lists the exact commands it runs. Use before running checks or saying work on the LADO repo is done, when a check fails, when you merge a run's branch, when you release LADO, or when you find a LADO bug or debt. `verification-before-completion` and `diagnosing-bugs` hold the general discipline.
---

# LADO checks

Run every command from the repository root. A claim is proven only by output you ran
yourself after your last change; quote it in your step's artifact (the name the step
`produces`, e.g. `report` or `verify`), not in its note.

Checks are slow (the whole unit suite takes about 4 minutes), so each role runs a fixed
list of commands, written in its own file: the developer the fast ones, the reviewer
`make lint`, and the checker, in the `verify` step before `merge_ok`, `make check` on the
branch merged with main. The role's list is the whole proof of its claims: "partial proves
nothing" (`verification-before-completion`) means no fewer than the list names, not more. In a kit's own repository (e.g. kit-lado-dev) there is
no Makefile: `lado kits check .` is the whole check, for every role.

## Running tests

`pyproject.toml` sets `addopts = -m 'not integration and not live and not smoke and not ui'`,
so a test file runs with its folder's `-m`: `tests/` none, `tests/integration/`
`-m integration`, `tests/ui/` `-m ui` (after `make web` and `make browser`),
`tests/smoke/` `-m smoke`. An integration or UI file run without its `-m` is silently
dropped: a `0 passed` or `no tests ran` line proves nothing, and in a mixed run `N passed`
hides the dropped files. Run each layer as its own command,
and add `-n auto` to every unit, integration and UI pytest command that runs a suite or
whole files: the Makefile passes it, and without it pytest runs one test at a time
(minutes instead of seconds). A rerun of single node ids (`file::test`) uses `-n0`.

`make check` runs `make lint`, `make test-js`, `make web` and `make browser`, then three
pytest runs one after another: unit, `-m integration`, `-m ui`. All three run even when one
fails; at the end it prints `check: failed: <groups>` and exits non-zero. To read a red
`make check`, find the `check: failed:` line, then each failed group's own pytest summary.

**Live tests** (`-m live`: `make test-live`, anything under `tests/live/`) drive real agent
CLIs and models, serially. They run only at a release ("Releasing LADO") or when the human
asks for them, and only the supervisor runs them, locally; no worker does. In this kit's
runs this replaces AGENTS.md's "and a provider's after changing it" (the human,
2026-10-10), so a change merged without a live run is no finding. A run with
`PROVIDER=claude`, or with no `PROVIDER` (every provider, Claude included), uses a paid
model and needs the human's yes first, every time.

When the human asks for them during a run (most likely at `merge_ok`, after the checker's
note that no live tests ran): run them in the run's worktree (`flow_status`) at the commit
`verify` checked, never while a check runs there. Write the output as an artifact of the
run (`<run>/live-tests`) and tell the human the command and its last summary line, or the
first failing line. A red result is no outcome of yours: the human decides, e.g. rejects
the merge at `merge_ok` quoting that line, which sends it to `implement`.

Never pipe a check whose result gates something (`make check | tail` hides a red exit
status): write it to a log outside the tree (`make check > <log> 2>&1`; a log in the
worktree is an uncommitted change), take the exit status, then read the log's last lines.
Never start a second long check while one runs; wait for its result.

## Releasing LADO

The supervisor, only when the human asks, in the order of LADO's AGENTS.md ("Release"):

1. `uv version <X.Y.Z>`, and the release date in place of "(unreleased)" in CHANGELOG.md:
   one commit on main, the only direct commit the request allows.
2. Check `gh auth status` (step 4 needs it; if it fails, tell the human), then push main
   with the human's yes. That yes also covers starting the Windows probe in 4. The commit
   you push is the release commit: step 1's, or after a fix, main with the fix merged.
3. CI green on that commit, the smoke on Linux and macOS with it (CI runs everything
   `make check` runs, so no separate `make check`).
4. The Windows probe green: `gh workflow run windows-shell.yml --ref main -f
   runner=windows-latest`, then `gh run watch <id> --exit-status`.
5. `make test-live` locally, on main at the release commit, for every provider, Claude
   with the human's yes; rerun a
   failure as "When a check fails" says for flaky. When the human declines Claude's run,
   run every other provider, one `make test-live PROVIDER=<name>` each; they must be green,
   and the human decides whether to release. A provider skipped for an environment cause
   (its CLI missing or not logged in) is not green: tell the human.
6. Tag the release commit (`git tag vX.Y.Z <the release commit>`) and push the tag, with
   the human's yes; it publishes to PyPI.

Red CI, probe or live tests: no tag. Classify the failure first ("When a check fails"):
for environment or flaky, tell the human and rerun once (a workflow with `gh run rerun
<id> --failed`); for code or test, tell the human, quoting the failing line. The fix goes
through `fix`; its CHANGELOG.md line goes under the dated `## X.Y.Z`, and it updates the
date if the day changed. The release goes on from step 2 with the same version. A
BACKLOG.md entry found during a release (a flaky test) is committed on main after the tag
is pushed, so main does not move past the release commit.

Done when the tag is pushed, or the human chose to stop and you told them what is left.

## Merging a run's branch

The supervisor's `merge` step comes after `verify` (the checker merged main into the run's
branch and found the full check green) and the human's `merge_ok`. It runs no check:

1. For each **Found on the way** item in the approving review and the green `verify` (the
   run's artifacts `<run>/review` and `<run>/verify`, read with `read_artifact`), add a
   BACKLOG.md entry (below) in the run's worktree and commit it on the
   run's branch (a commit of BACKLOG.md alone needs no new check).
2. In your repo, on main, `git merge --ff-only <the run's branch>`. If it fails because
   main moved since `verify`, report `stale`: the run goes back to `verify`, which merges
   the new main and checks again; after its `green` the human answers Merge? again. Any
   other failure (local changes, another branch checked out, …) is no outcome: show the
   human the command and its output, and retry once they say it is settled. Never stash or
   reset their checkout.

Done when each item found on the way has its entry and main is at the run's branch. Then
report `merged`.

## When a check fails

Only the developer fixes; a reviewer and the checker report the class (the checker as its
role says).

1. Find the first real error and quote that exact line, not the summary.
2. Classify it:
   - **code**: the product does the wrong thing. Fix the code.
   - **test**: the test expects the wrong thing or depends on order or timing. Fix the
     test and say why it was wrong.
   - **flaky**: passes on rerun with no change. Rerun only what failed, up to 2 times
     (the checker: what to rerun after a red `make check` is in its role, section 2). If it
     passes, report it as flaky with the failing line, and record it in BACKLOG.md (the
     checker lists it under **Found on the way**). If it fails every time, it is not
     flaky: treat it as code or test.
   - **environment**: tmux, node, uv, a CLI login, a network or disk problem. Report what is
     missing and the command that showed it; do not change code to work around it. The
     developer reports BLOCKED; the reviewer and the checker tell the supervisor and leave
     the step open.
3. Never mark a test skipped or loosen an assertion to get green.

Done when: the failure has a class, a quoted line and, for code or test, a fix with the
check green again.

## Recording a bug, friction or debt in BACKLOG.md

Record what you find outside your task instead of fixing it silently or working around it:
a bug, a friction (a missing option, a confusing message), or debt (a design that a coming
ROADMAP stage will have to undo, a patch over a root cause, a second source of truth).
Add a section to `BACKLOG.md`:

```markdown
## <short title: what is wrong>

Size: <S|M|L>. Why here: <reason>.
<What happens: the command or step, what you saw.>
Wanted: <what should happen instead.>
Found: <YYYY-MM-DD>, <context: task or check where it showed up>.
```

Keep it to a few lines; check first that no entry covers it already. Add it at the end of
the tier it belongs to (P0–P3, as BACKLOG.md's header says): `.gitattributes` merges
BACKLOG.md with `merge=union`, so entries that parallel branches add merge without a
conflict. Mention the new entry in your report.

Who writes the entry: the next agent that writes on the run's branch. In a flow run the
design, the read-only architect and reviewer, and the checker list what they found under
**Found on the way** in their step's artifact (`design`, `architecture-review`, `review`,
`verify`), each item as title, what happens, what is wanted. The design carries
its own and the architect's items to the developer, who adds the entries in `implement`,
together with the reviewer's items of a `changes` review and the checker's of a `red` or
`conflict`. The supervisor adds those of the review that approved the branch and of the
green `verify` in `merge`. When it cancels a run, at any step, it adds on main every
**Found on the way** item of the run that has no entry on main yet, since the run's branch
is not merged; it tells the human that the run's worktree and branch are kept, and removes
them only on their yes. Outside a run, the supervisor adds them on main.

Done when: each item found has a BACKLOG.md entry, committed on the run's branch or on
main, and the report names it.

## Clean-room

LADO borrows ideas, never code. Do not copy code, tests, prompt texts or file layouts from
other projects, including the agent CLIs and orchestrators you look at for ideas. Describe
the behaviour in your own words first, then write it from scratch.
