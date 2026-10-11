# Backlog: lado-dev

## 0.13.1: simplify a red live run the human asked for at `merge_ok`

Size: M. Why here: left open at the 0.13.0 release by the human's choice (run `improve/lado-dev`, `release_ok`, 2026-10-11).
The path "the human asks for live tests at `merge_ok`, they are red, the branch goes back to `implement`" has 5 medium findings in `kit-reports/lado-dev-0.13.0-2026-10-11.md` (visit 3): F5.1 (the `implement` done condition has no exception for an AC marked "needs a live run"), F4.1 (the reviewer accepts that mark on any AC), F3.4 (no flaky or environment rule for a requested live run before its red result), F3.1 (an AC only a live run can show has no path on the first `implement` visit, nor in a release's `fix` run), F5.2 (a fix with no reproduction clashes with `diagnosing-bugs`); and 1 low, F3.5 (a CI or probe run green on its rerun is not recorded as flaky).
Wanted: a simpler shape rather than more rules, as the critic suggests: a red live run asked for at `merge_ok` is the human's to settle (reject with the failing line; the developer treats it as any failing test it can reproduce, or reports BLOCKED), with no "needs a live run" mark for the reviewer to weigh. Also restate BLUEPRINT.md R8 ("No role runs live tests in a run" → no worker does; the supervisor runs them at `merge_ok` when the human asks, R9), the §1 header "(live tests only at a release)", and R9's rerun order as the kit has it (CI or probe rerun once, live by the flaky rule; red every time is classified).
Found: 2026-10-11, evaluation visit 3 of run `improve/lado-dev`.

## kit-builder: `flow_diagram.py --compare` rejects `produces`/`reads` in skeletons

Size: S. Why here: every lado-dev round compares against a stripped copy of BLUEPRINT.md.
`flow_diagram.py . --compare BLUEPRINT.md` exits 2: "a skeleton holds only agent, gate, end, outcomes, max_visits (not produces)".
Wanted: the skeleton format accepts and compares `produces` and `reads` (LADO 0.27+). To report to kit-builder.
Found: 2026-10-11, assess and evaluate of run `improve/lado-dev`.
