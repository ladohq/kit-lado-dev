---
name: lado-checks
description: Which LADO check proves which claim, how to read a failure, and how to record a bug or friction in BACKLOG.md. Use before saying work on the LADO repo is done, when a check fails, or when you hit a LADO bug.
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
| The change is ready | `make check` | Always, last, before you report done. It runs all four above. |
| A real agent CLI still works | `make test-live PROVIDER=claude` or `PROVIDER=kilo` | After changing a provider, and before a release. Ask the supervisor first for Claude: it uses a paid model. |

Done when: the report names each command you ran and its last summary line (for pytest,
the `N passed` line).

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

## Recording a bug or friction in BACKLOG.md

When LADO itself gets in your way (a bug, a missing option, a confusing message), add a
section to `BACKLOG.md` instead of working around it silently:

```markdown
## <short title: what is wrong>

<What happens: the command or step, what you saw.>
Wanted: <what should happen instead.>
Found: <YYYY-MM-DD>, <context: task or check where it showed up>.
```

Keep it to a few lines. Mention the new entry in your report.

## Clean-room

LADO borrows ideas, never code. Do not copy code, tests, prompt texts or file layouts from
other projects, including the agent CLIs and orchestrators you look at for ideas. Describe
the behaviour in your own words first, then write it from scratch.
