---
name: lado-checks
description: Which LADO checks to run for a change (by changed path) and who runs them, how the merge step merges a run's branch with the full check, how to read a failure, and how to record a bug, friction or debt in BACKLOG.md and who records it. Use before running checks or saying work on the LADO repo is done, when a check fails, when you merge a run's branch, when you release LADO, or when you find a LADO bug or debt. It names the LADO commands and rules; `verification-before-completion` and `diagnosing-bugs` hold the general discipline.
---

# LADO checks

Run every command from the repository root. A claim is proven only by output you ran
yourself after your last change; quote it.

Checks are slow (the whole unit suite takes about 4 minutes), so while a task is in work
only the checks that cover the changed files run; the full `make check` runs only, and
always, in the merge step, on the branch merged with main.

## What to check for a change

This section is for the LADO repository. In a kit's own repository (e.g. kit-lado-dev)
there is no Makefile: `lado kits check .` is its only check, during work and at merge.

The changed files are `git diff --name-only main...HEAD` plus uncommitted ones. Always run
`make lint` (`make fmt` fixes most of it), then for each changed path every row that
matches it:

| Changed path | Run |
|---|---|
| `src/lado/**.py` | each `test_*.py` the search below finds, with its folder's `-m` |
| A `test_*.py` file outside `tests/live/` | that file, with its folder's `-m` |
| A `conftest.py` or helper under `tests/`, outside `tests/live/` | each `test_*.py` in its folder, with its `-m`; for one directly in `tests/`, `make test` |
| `web/` | `make web` (also fails on a stale `web/openapi.json`), `make browser`, then the UI tests of each screen it changes, by file name (`tests/ui/test_<screen>.py`); for a file that names no screen (e.g. `Shell.tsx`, `App.tsx`, `styles.css`, `tokens.css`, `api.ts`, `live.ts`, `Settings.tsx`, `Team.tsx`, `Sessions.tsx`), all of `tests/ui/` with `-m ui` |
| `src/lado/providers/opencode_plugin.js`, `tests/js/` | `make test-js` |
| `tests/live/` | the live tests, as below |
| Only non-code paths: BACKLOG.md, ROADMAP.md, README.md, AGENTS.md, CLAUDE.md, `docs/` | nothing more. Any other path is code, also a `.md` under `src/` or `tests/` |
| A path no row matches | `make test` (all unit tests) |

**The tests that import a module.** For `src/lado/<mod>.py` set `P=lado`; for
`src/lado/<pkg>/<mod>.py` set `P=lado.<pkg>`, and run the search a second time with `P=lado`
and `<mod>` set to `<pkg>`: many tests reach a module through its package (`from lado
import providers`, then `providers.get("claude")`).

```bash
grep -rlE "^\s*(from $P import .*\b<mod>\b|(from|import) $P\.<mod>\b)" tests --include='*.py' | grep -v '^tests/live/'
```

Only `test_*.py` hits run; a hit in a `conftest.py` or a helper pulls in nothing. Each runs
with its folder's `-m`: `tests/` none, `tests/integration/` `-m integration`, `tests/ui/`
`-m ui` (after `make web` and `make browser`). When a changed module has no `test_*.py`
hit, or more than 10 unit hits (both searches together), run `make test` in place of its
unit hits. Integration and UI tests run only as hits, and with none, none run: the full
`make check` at merge covers them, so a short rule both developer and reviewer apply alike
beats a complete one.

`pyproject.toml` sets `addopts = -m 'not integration and not live and not ui'`, so an
integration or UI file run without its `-m` is silently dropped: a `0 passed` or `no tests
ran` line proves nothing, and in a mixed run `N passed` hides the dropped files. Run each
layer as its own command, and add `-n auto` to every unit, integration and UI pytest
command: the Makefile passes it, and without it pytest runs one test at a time (minutes
instead of seconds). Live tests run serially, through `make test-live`.

**Live tests** (`-m live`: `make test-live`, anything under `tests/live/`) drive real agent
CLIs and models. They run only when the change touches `src/lado/providers/`,
`src/lado/hooks.py`, `src/lado/mcp_server.py`, `src/lado/runtime.py`,
`src/lado/agent_env.py` or `tests/live/`, once per round, by the developer, after the
checks above, and at a release. A change to one provider runs only that provider's
(`PROVIDER=<name>`). A run with `PROVIDER=claude`, or with no `PROVIDER` (every provider,
Claude included), uses a paid model: whoever runs it, the developer through the supervisor
or the supervisor at a release, gets the human's yes first, every time, also inside a run.
On a no, run only the other providers the change needs, one `make test-live
PROVIDER=<name>` each, or none; the developer's report says "the human declined Claude's
run" (DONE_WITH_CONCERNS). Live tests are green only when each provider they need passed:
a skipped provider is not green. One skipped for an environment cause (its CLI missing or
not logged in, "When a check fails"): the developer reports DONE_WITH_CONCERNS naming it.

## Who runs what

The checks above are the whole proof of a claim while a task is in work: "partial proves
nothing" (`verification-before-completion`) means no fewer than they name, not the full
set, which runs at merge.

- **Developer**: after your last change, the checks above for your change, and
  nothing more; after a `red` from the merge step, also the failing checks its note quotes
  (a test with its `-m`, or the make target); live tests as above. Your report names each
  command and its last summary line (for pytest, the `N passed` line).
- **Reviewer**: on the reviewed commit, run the same checks yourself, and check that the
  developer's report ran every one the changed paths need. A missing or wrong
  check is a finding: Important when your own run of it is red for code or test,
  otherwise Minor; a check that cannot run for an environment cause is no finding ("When a
  check fails"). Do not run live tests: check that the report shows them green on the
  reviewed commit when the rule above needs them, or name their absence as an Important
  finding, unless the report says the human declined Claude's run or names a provider
  skipped for an environment cause: then that absence is a concern for `merge_ok`, not a
  finding. Name the commit your checks ran on.
- **Merge step**: `make check`, always (below).
- **Release** (the supervisor, only when the human asks): the request allows one direct
  commit on main, the version bump (`uv version <X.Y.Z>`, commit). Then, in this order:
  push main; wait for green CI on that commit (CI runs everything `make check` runs, so no
  separate `make check`); the live tests of every provider (`make test-live`); push the tag
  `vX.Y.Z`, which publishes to PyPI. The tag needs green CI and green live tests on that
  commit. When the human declines Claude's run, run every other provider, one `make
  test-live PROVIDER=<name>` each; they must be green, and the human decides whether to
  release. Red CI or live tests: no tag; tell the human, quoting the failing line; the fix
  goes through `fix`, then the release goes on from pushing main with the same version.
  Each push needs the human's yes.

`make check` runs `make lint`, `make test-js`, `make web` and `make browser`, then one
parallel pytest run of the unit, integration and UI tests (`-m 'not live'`).

Never pipe a check whose result gates something (`make check | tail` hides a red exit
status): write it to a log outside the tree (`make check > <log> 2>&1`; a log in the
worktree is an uncommitted change), take the exit status, then read the log's last lines.
Never start a second long check while one runs; wait for its result.

## Merging a run's branch

The supervisor's `merge` step records what the approving review (in the step's note) found
on the way, then checks the branch together with the current main before main moves:

1. For each **Found on the way** item in the review, add a BACKLOG.md entry (below) in the
   run's worktree and commit it on the run's branch.
2. In the run's worktree, `git merge main`. If it conflicts, `git merge --abort` and report
   `conflict`, with the conflicting files in the note.
3. Run `make check` (in a kit's own repository, `lado kits check .`). If it is red, rerun
   the whole check once before classifying; this rerun takes the place of the reruns in
   "When a check fails". Green: it was flaky; add its BACKLOG.md entry on the run's branch
   and go on (a commit of BACKLOG.md alone after a green check needs no new check). Red
   again: classify the second run's first error, as code, test or environment ("When a
   check fails"). Code or test: report `red`, with the failing output in the note; the
   task goes back to `implement`.
   Environment: tell the human what is missing and leave the step open.
4. In your repo, on main, `git merge --ff-only <the run's branch>`. If main moved in the
   meantime and that fails, start again at 2.

Done when each item found on the way has its entry, main is at the run's branch and the
check of step 3 was green on it, or on its parent when the last commit only adds
BACKLOG.md entries. Then report `merged`.

## When a check fails

Only the developer fixes; a reviewer reports the class, the merge step acts as its step 3
says.

1. Find the first real error and quote that exact line, not the summary.
2. Classify it:
   - **code**: the product does the wrong thing. Fix the code.
   - **test**: the test expects the wrong thing or depends on order or timing. Fix the
     test and say why it was wrong.
   - **flaky**: passes on rerun with no change. Rerun up to 2 times. If it passes, report it
     as flaky with the failing line, and record it in BACKLOG.md. If it fails 3 times, it is
     not flaky: treat it as code or test.
   - **environment**: tmux, node, uv, a CLI login, a network or disk problem. Report what is
     missing and the command that showed it; do not change code to work around it. The
     developer reports BLOCKED; only a live provider skipped for it is DONE_WITH_CONCERNS
     ("Live tests").
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
design and the read-only roles (architect, reviewer) list what they found under **Found on
the way**, each item as title, what happens, what is wanted. The design carries its own and
the architect's items to the developer, who adds the entries in `implement`, together with
the reviewer's items of a `changes` review. The supervisor adds those of the review that
approved the branch in `merge`. When it cancels a run, at any step, it adds on main every
**Found on the way** item of the run that has no entry on main yet, since the run's branch
is not merged; it tells the human that the run's worktree and branch are kept, and removes
them only on their yes. Outside a run, the supervisor adds them on main.

Done when: each item found has a BACKLOG.md entry, committed on the run's branch or on
main, and the report names it.

## Clean-room

LADO borrows ideas, never code. Do not copy code, tests, prompt texts or file layouts from
other projects, including the agent CLIs and orchestrators you look at for ideas. Describe
the behaviour in your own words first, then write it from scratch.
