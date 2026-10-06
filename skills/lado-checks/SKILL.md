---
name: lado-checks
description: Which LADO checks to run for a change (by changed path) and who runs them, how the merge step merges a run's branch with the full check, how to read a failure, and how to record a bug, friction or debt in BACKLOG.md and who records it. Use before running checks or saying work on the LADO repo is done, when a check fails, when you merge a run's branch, or when you find a LADO bug or debt. It names the LADO commands and rules; `verification-before-completion` and `diagnosing-bugs` hold the general discipline.
---

# LADO checks

Run every command from the repository root. A claim is proven only by output you ran
yourself after your last change; quote it.

Checks are slow (the whole unit suite takes about 4 minutes), so while a task is in work
only the checks that cover the changed files run; the full `make check` runs only, and
always, in the merge step, on the branch merged with main.

## What to check for a change

The changed files are `git diff --name-only main...HEAD` plus uncommitted ones. Run
`make lint` (`make fmt` fixes most of it), then for each changed path every row that
matches it:

| Changed path | Run |
|---|---|
| `src/lado/<mod>.py`, `src/lado/<pkg>/<mod>.py` | `uv run pytest tests/test_<mod>.py` and each test file that imports the module (`grep -rlw <mod> tests/`) |
| Code that drives processes, tmux, git, hooks, `lado mcp`, the server process or a provider (`runtime.py`, `hooks.py`, `mcp_server.py`, `tmux.py`, `state.py`, `loop.py`, `server/`, `providers/`) | also the matching `uv run pytest -m integration tests/integration/test_<x>.py` (fake agent, no LLM) |
| `web/`, or the server's API or UI (`src/lado/server/`) | `make web` (also fails on a stale `web/openapi.json`), `make browser` once, then `uv run pytest -m ui tests/ui/test_<screen>.py` for each screen it touches |
| `src/lado/providers/opencode_plugin.js`, `tests/js/` | `make test-js` |
| A test file | that file, with its `-m` |
| Only non-code paths: BACKLOG.md, ROADMAP.md, README.md, AGENTS.md, CLAUDE.md, `docs/` | `make lint` only. Any other path is code, also a `.md` under `src/` or `tests/` |
| A kit, in its own repository (e.g. kit-lado-dev) | `lado kits check .` |
| Code no row above matches | `make test` (all unit tests) |

`pyproject.toml` sets `addopts = -m 'not integration and not live and not ui'`: a file under
`tests/integration/` or `tests/ui/` run without its `-m integration` or `-m ui` silently
collects zero tests. A `0 passed` or `no tests ran` line proves nothing.

**Live tests** (`make test-live PROVIDER=claude|kilo|opencode`: real agent CLIs and
models) run only when the change touches a provider, hooks, the MCP server or how agents
get their input, once per round, by the developer, after the checks above. For
`PROVIDER=claude` the developer asks the supervisor and waits for the human's yes, every
time, also inside a run: it uses a paid model. On a no, the report says the live check did
not run (DONE_WITH_CONCERNS).

## Who runs what

- **Developer**: after your last change, the checks the table names for your change, and
  nothing more; live tests as above. Your report names each command and its last summary
  line (for pytest, the `N passed` line).
- **Reviewer**: on the reviewed commit, run the same rows of the table yourself, and check
  that the developer's report ran every row the changed paths need; a missing or wrong
  check is a finding. Do not run live tests: check that the report shows them green on
  the reviewed commit when the rule above needs them, or name their absence as a finding.
  Name the commit your checks ran on.
- **Merge step**: `make check`, always (below).
- **Release**: no separate `make check`; CI runs everything `make check` runs on the
  pushed commit. A release needs green CI on the release commit and `make test-live` on
  main.

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
3. Run `make check`. If it is red, classify the failure as "When a check fails" says.
   Report `red`, with the failing output in the note, only for code or test: the task goes
   back to `implement`. For environment, tell the human what is missing and leave the step
   open.
4. In your repo, on main, `git merge --ff-only <the run's branch>`. If main moved in the
   meantime and that fails, start again at 2.

Done when each item found on the way has its entry, main is at the run's branch and
`make check` was green on it. Then report `merged`.

## When a check fails

1. Find the first real error and quote that exact line, not the summary.
2. Classify it:
   - **code**: the product does the wrong thing. Fix the code.
   - **test**: the test expects the wrong thing or depends on order or timing. Fix the
     test and say why it was wrong.
   - **flaky**: passes on rerun with no change. Rerun up to 2 times. If it passes, report it
     as flaky with the failing line, and record it in BACKLOG.md. If it fails 3 times, it is
     not flaky: treat it as code or test.
   - **environment**: tmux, node, uv, a CLI login, a network or disk problem. Report what is
     missing and the command that showed it; do not change code to work around it.
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

<What happens: the command or step, what you saw.>
Wanted: <what should happen instead.>
Found: <YYYY-MM-DD>, <context: task or check where it showed up>.
```

Keep it to a few lines; check first that no entry covers it already. Add it at the end of
the file: `.gitattributes` merges BACKLOG.md with `merge=union`, so entries that parallel
branches append merge without a conflict. Mention the new entry in your report.

Who writes the entry: the next agent that writes on the run's branch. In a flow run the
design and the read-only roles (architect, reviewer) list what they found under **Found on
the way**, each item as title, what happens, what is wanted. The design carries its own and
the architect's items to the developer, who adds the entries in `implement`, together with
the reviewer's items of a `changes` review. The supervisor adds those of the review that
approved the branch in `merge`. When it cancels a run, at any step, it adds on main every
**Found on the way** item of the run that has no entry on main yet, since the run's branch
is removed with it. Outside a run, the supervisor adds them on main.

Done when: each item found has a BACKLOG.md entry, committed on the run's branch or on
main, and the report names it.

## Clean-room

LADO borrows ideas, never code. Do not copy code, tests, prompt texts or file layouts from
other projects, including the agent CLIs and orchestrators you look at for ideas. Describe
the behaviour in your own words first, then write it from scratch.
