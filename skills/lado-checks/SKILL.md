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

This section is for the LADO repository. In a kit's own repository (e.g. kit-lado-dev)
there is no Makefile: `lado kits check .` is its only check, during work and at merge.

The changed files are `git diff --name-only main...HEAD` plus uncommitted ones. Run
`make lint` (`make fmt` fixes most of it), then for each changed path every row that
matches it:

| Changed path | Run |
|---|---|
| `src/lado/**.py` | The test files that import the module (below), each with its layer's `-m` |
| Code that drives processes, tmux, git, hooks, `lado mcp`, the server process or a provider (`runtime.py`, `hooks.py`, `mcp_server.py`, `tmux.py`, `state.py`, `loop.py`, `server/`, `providers/`) | also the integration tests: those the search below finds, or all of them (`uv run pytest -m integration`) when it finds none |
| `web/`, `src/lado/server/` | `make web` (also fails on a stale `web/openapi.json`), `make browser` once, then the UI tests (`uv run pytest -m ui tests/ui/test_<screen>.py`) of each screen it changes, by file name; all of `tests/ui/` when unsure |
| `src/lado/providers/opencode_plugin.js`, `tests/js/` | `make test-js` |
| A test file outside `tests/live/` | that file, with its layer's `-m` |
| `tests/live/` | the live tests, as below |
| Only non-code paths: BACKLOG.md, ROADMAP.md, README.md, AGENTS.md, CLAUDE.md, `docs/` | `make lint` only. Any other path is code, also a `.md` under `src/` or `tests/` |
| Anything else: a path no row matches, a module the search finds no test file for or more than 10 unit test files for | `make test` (all unit tests) in place of the unit hits; integration and UI hits still run with their `-m` |

**The tests that import a module.** For `src/lado/<mod>.py` set `P=lado`, for
`src/lado/<pkg>/<mod>.py` set `P=lado.<pkg>`, then run the search below. For a module in a
package, also run it with `P=lado` and `<mod>` set to `<pkg>`: many tests reach a module
through its package (`from lado import providers`, then `providers.get("claude")`).

```bash
grep -rlE "^\s*(from $P import .*\b<mod>\b|(from|import) $P\.<mod>\b)" tests --include='*.py' | grep -v '^tests/live/'
```

A hit that is not a `test_*.py` file (a `conftest.py`, a helper) stands for every test file
in its folder; `tests/conftest.py` stands for the unit files in `tests/` only. A hit on
`tests/ui/conftest.py` or `tests/integration/conftest.py` does not pull in its whole layer:
UI tests run only when a path matches the `web/`, `src/lado/server/` row, integration tests
only through the row for processes, tmux, hooks, `lado mcp` and providers, and then only
the test files the search finds. Run each hit with its folder's layer: `tests/` plain,
`tests/integration/` with `-m integration`, `tests/ui/` with `-m ui` (after `make web` and
`make browser`).
`pyproject.toml` sets `addopts = -m 'not integration and not live and not ui'`, so an
integration or UI file run without its `-m` is silently dropped: a `0 passed` or `no tests
ran` line proves nothing, and in a mixed run `N passed` hides the dropped files. Run each
layer as its own command, and add `-n auto` to every unit, integration and UI pytest
command: the Makefile passes it, and without it pytest runs one test at a time (minutes
instead of seconds). Live tests run serially, through `make test-live`.

**Live tests** (`-m live`: `make test-live`, anything under `tests/live/`) drive real agent
CLIs and models. They run only when the change touches a provider, hooks, the MCP server,
how agents get their input or `tests/live/`, once per round, by the developer, after the
checks above, and at a release. A change to one provider runs only that provider's
(`PROVIDER=<name>`). A run with `PROVIDER=claude`, or with no `PROVIDER` (every provider,
Claude included), uses a paid model: whoever runs it, the developer through the supervisor
or the supervisor at a release, gets the human's yes first, every time, also inside a run.
On a no, run the other providers' live tests; the developer's report says "the human
declined Claude's run" (DONE_WITH_CONCERNS). Live tests are green only when each provider
they need passed: a skipped provider is not green; say why (environment, "When a check
fails").

## Who runs what

The rows of the table are the whole proof of a claim while a task is in work: "partial
proves nothing" (`verification-before-completion`) means no fewer than the table names, not
the full set, which runs at merge.

- **Developer**: after your last change, the checks the table names for your change, and
  nothing more; after a `red` from the merge step, also the failing checks its note quotes
  (a test with its `-m`, or the make target); live tests as above. Your report names each
  command and its last summary line (for pytest, the `N passed` line).
- **Reviewer**: on the reviewed commit, run the same rows of the table yourself, and check
  that the developer's report ran every row the changed paths need. A missing or wrong
  check is a finding: Important when your own run of it is red for code or test,
  otherwise Minor; a check that cannot run for an environment cause is no finding ("When a
  check fails"). Do not run live tests: check that the report shows them green on the
  reviewed commit when the rule above needs them, or name their absence as an Important
  finding, unless the report says the human declined Claude's run: then Claude's absence is
  a concern for `merge_ok`, not a finding. Name the commit your checks ran on.
- **Merge step**: `make check`, always (below).
- **Release**: no separate `make check`; CI runs everything `make check` runs on the
  pushed commit. A release needs green CI and green live tests on the release commit, or
  the human, having declined Claude's run, decided to release: then the other providers'
  live tests run and must be green.

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
3. Run `make check` (in a kit's own repository, `lado kits check .`). If it is red,
   classify the failure as "When a check fails" says. Report `red`, with the failing output
   in the note, only for code or test: the task goes back to `implement`. For flaky, rerun
   the whole `make check` once: green, go on and add a BACKLOG.md entry for the flake on
   the run's branch; red again, it is code or test. For environment, tell the human what
   is missing and leave the step open.
4. In your repo, on main, `git merge --ff-only <the run's branch>`. If main moved in the
   meantime and that fails, start again at 2.

Done when each item found on the way has its entry, main is at the run's branch and the
check of step 3 was green on it. Then report `merged`.

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
the tier it belongs to (P0–P3, as BACKLOG.md's header says), with its `Size:` and `Why
here:` line: `.gitattributes` merges BACKLOG.md with `merge=union`, so entries that
parallel branches add merge without a conflict. Mention the new entry in your report.

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
