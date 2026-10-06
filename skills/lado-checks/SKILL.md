---
name: lado-checks
description: Which LADO check proves which claim, how to read a failure, how the merge step merges a run's branch, and how to record a bug, friction or debt in BACKLOG.md and who records it. Use before saying work on the LADO repo is done, when a check fails, when you merge a run's branch, or when you find a LADO bug or debt. It names the LADO commands and rules; `verification-before-completion` and `diagnosing-bugs` hold the general discipline.
---

# LADO checks

Run every command from the repository root. A claim is proven only by output you ran
yourself after your last change; quote it.

## Which command proves what

| Claim | Command | When |
|---|---|---|
| Code is formatted and lint-clean | `make lint` | After any Python change. `make fmt` fixes most of it. |
| Pure logic works | `make test` | After any change; fastest feedback. |
| Behaviour across processes works (tmux, git, hooks, `lado mcp`, SQLite) | `make test-integration` | After touching `runtime.py`, `hooks.py`, `mcp_server.py`, `tmux.py`, `state.py` or a provider. Uses the fake agent, no LLM. |
| The Kilo plugin works | `make test-js` | After touching `kilo_plugin.js` or its tests. |
| The change is ready | `make check` | Developer: last, before you report done, except a change only to non-code paths (below), which needs `make lint` only. Reviewer and merge step: as the rules below say. It runs all four above. |
| A real agent CLI still works | `make test-live PROVIDER=claude` or `PROVIDER=kilo` | Developer: after `make check`, when the rule below says. Release: on main. For Claude, ask the human, through the supervisor, every time: it uses a paid model. |

Done when: the report names each command you ran and its last summary line (for pytest,
the `N passed` line).

## Run as little as proves the claim

Checks are slow; run each one once, when it proves something new.

- While working, run only the tests next to your change (`uv run pytest tests/test_x.py -k
  name`); run the final check below once, after your last change, before you report.
- A change only to non-code paths (defined below) needs `make lint` only; a change to a
  kit (its own repository, e.g. kit-lado-dev) needs `lado kits check`. Any other change needs `make check`.
- `make test-live` runs only when the change touches a provider, hooks, the MCP server or
  how agents get their input, and only once per round: the developer runs it and reports
  it. For `PROVIDER=claude` it asks the supervisor and waits for the human's yes; on a no,
  its report says the live check did not run (DONE_WITH_CONCERNS). A reviewer does not
  run it: it checks that the developer's report shows it green on the reviewed commit, or
  names its absence as a finding, and reruns `make check` only.
- **Non-code paths** are BACKLOG.md, ROADMAP.md, README.md, AGENTS.md, CLAUDE.md and
  anything under `docs/`. Every other path is code, also a `.md` under `src/` (`make check`
  validates the built-in kits) or under `tests/`.
- A reviewer runs `make check` once per review and names the commit it ran on in its
  review. On a re-review it keeps that result only when every path in
  `git diff --name-only <that commit> HEAD` is a non-code path; otherwise it runs it again.
- The merge step skips `make check` only when both hold: `git merge main` brought nothing
  (`git rev-parse HEAD` is the same before and after it; git's "Already up to date" is
  only a hint, its wording depends on the version and locale), and the approving review
  names the commit its green `make check` ran on and `git log --name-only <that
  commit>..HEAD` shows only non-code paths (such as the merge step's BACKLOG.md commit).
  Then it says which commit's check stands. Otherwise it runs `make check`.
- A release runs no separate `make check`: main was checked at each merge and CI checks
  the pushed commit. It needs `make test-live` on main and green CI on the release commit.
- Never pipe a check whose result gates something (`make check | tail` hides a red exit
  status): write it to a log outside the tree (`make check > <log> 2>&1`; a log in the
  worktree is an uncommitted change), take the exit status, then read the log's last lines.
- Never start a second long check while one runs; wait for its result.

## Merging a run's branch

The supervisor's `merge` step records what the approving review (in the step's note) found
on the way, then checks the branch together with the current main before main moves:

1. For each **Found on the way** item in the review, add a BACKLOG.md entry (below) in the
   run's worktree and commit it on the run's branch.
2. In the run's worktree, note `git rev-parse HEAD`, then `git merge main`. If it
   conflicts, `git merge --abort` and report `conflict`, with the conflicting files in the
   note.
3. Run `make check` unless the merge-step rule in "Run as little as proves the claim"
   lets you skip it. If it is red, classify the failure as "When a check fails" says.
   Report `red`, with the failing output in the note, only for code or test; for
   environment, tell the human what is missing and leave the step open.
4. In your repo, on main, `git merge --ff-only <the run's branch>`. If main moved in the
   meantime and that fails, start again at 2.

Done when each item found on the way has its entry, main is at the run's branch and the
code that lands has a green `make check`: yours, or the reviewer's when step 3 let you
skip it. Then report `merged`.

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
