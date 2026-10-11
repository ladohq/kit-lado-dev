# Kit report: lado-dev 0.13.0

- Date: 2026-10-11
- Kit: the worktree of run `improve/lado-dev` (`.lado/worktrees/kit-lado-dev/improve-lado-dev`), branch `lado/kit-lado-dev/improve-lado-dev`; path given by the task
- Commit: `d673619` (kit text of round 4 `13dd8d5`, blueprint `d673619`)
- Evaluated by: kit-builder critic (layers a and b)
- Mode: re-evaluation, visit 3 of step `evaluate` in `improve`; verdict asked.
  - The previous report is this file at `166c46a` (visit 2); git keeps it. The base the plan names is `38324bf`.
  - This visit reviews round 4: `git diff 166c46a -- kit.yaml README.md BLUEPRINT.md agents flows skills`. It touches 4 files: `BLUEPRINT.md`, `agents/developer.md`, `agents/reviewer.md` and `skills/lado-checks/SKILL.md`. The whole run's change (`git diff 38324bf`) was given as context.
- Passes: three independent sub-agents went over round 4's diff and its surroundings.
  - Pass A started from the roles.
  - Pass B started from the flows and traced these cases:
    - a request for live tests during `review`;
    - red live tests at `merge_ok`, a reject, then `implement` and `review`;
    - a first-visit AC that only a live run shows;
    - a release with red CI, a red probe, flaky Claude reruns and a code fix.
  - Pass C started from `lado-checks` against LADO's AGENTS.md and workflows. It confirmed that `gh run rerun <id> --failed` fits the `workflow_dispatch` probe and keeps its commit.
  - A fourth sub-agent made the full pass over the whole of the 4 touched files.
  - The critic checked every quote with `grep -nF`.
  - Dropped: 1 one-pass finding. It said step 1's "the only direct commit" conflicts with the BACKLOG.md commit after the tag; `agents/supervisor.md:70-71` allows BACKLOG.md writes on main.
  - Kept as confirmed: 1 one-pass finding (F3.5) and 2 full-pass findings (F3.4, F5.2).

Findings are candidates for the human to weigh, not a pass/fail grade.

## Card

| Layer | Result |
|---|---|
| a. `lado kits check` | OK; 0 warnings; with `--tag v0.13.0` also OK |
| a. Budget | Yellow: worker roles 4, work steps in `feature` 6, developer 1077 words, reviewer 857 words, 5 pairs of similar paragraphs (100%, 100%, 93%, 74%, 62%). Each yellow has its reason in BLUEPRINT.md §4; the reviewer's reason has been restated (old F9.1). The lead is at 947 (green). |
| a. Flows | 2 drawn; the flows did not change. `--compare BLUEPRINT.md` exits 2 on the skeletons' `produces`/`reads` (a limit of kit-builder's script). On a copy without them, the flows are the same as the skeletons; `produces`/`reads` agree when read by hand. |
| b. Rubric | 6 findings in the changed text (0 high, 5 medium, 1 low), plus 2 "Not traced" items (low). 9 of 12 criteria have no finding (1, 2, 6, 7, 8, 9, 10, 11, 12); the findings are under 3, 4 and 5. Of visit 2's 8 findings, 7 are RESOLVED and F3.1 is STILL OPEN for its first-visit case. Of its "Not traced" item, R9 is RESOLVED and R8 is STILL OPEN. |
| Covers | Re-evaluation of round 4's changed text, with a full pass over the 4 files it touches. |
| Stop rule | Not met: 5 medium findings in the changed text. This is advice for the release gate, not a block. |

The count covers only what the row "Covers" names: a count of a re-evaluation and one of a
full evaluation are not comparable.

### `lado kits check .`

```
lado-dev: OK (4 agents, 20 skills, 4 packs and 2 flows)
read-only roles: architect, reviewer
expects commands: git, make, uv, node, npm
```

`lado kits check . --tag v0.13.0` printed the same three lines (exit 0).

Expects: `expects commands: git, make, uv, node, npm`, with no warning.

Read-only: `read-only roles: architect, reviewer`. `dependencies.lado: ">=0.34"`, which is what `read_only` needs (checked by hand).

### Budget script (exit status 0)

```
# Complexity budget: lado-dev 0.13.0

| Measure | Where | Value | Green / yellow up to | Zone |
|---|---|---|---|---|
| Worker roles (not supervisor) | kit | 4 | 3 / 5 | yellow |
| Work steps in a flow | flows/feature.yaml | 6 | 5 / 8 | yellow |
| Work steps in a flow | flows/fix.yaml | 4 | 5 / 8 | green |
| Gates in a flow | flows/feature.yaml | 2 | 2 / 3 | green |
| Gates in a flow | flows/fix.yaml | 1 | 2 / 3 | green |
| Words in a role prompt | agents/architect.md | 587 | 800 / 1500 | green |
| Words in a role prompt | agents/checker.md | 555 | 800 / 1500 | green |
| Words in a role prompt | agents/developer.md | 1077 | 800 / 1500 | yellow |
| Words in a role prompt | agents/reviewer.md | 857 | 800 / 1500 | yellow |
| Words in the lead's prompt | agents/supervisor.md | 947 | 1000 / 1500 | green |
| Own skills | kit | 1 | 5 / 10 | green |
| MCP servers | kit | 0 | 2 / 4 | green |

## Similar paragraphs (one rule, one place; 55% similar or more)

- yellow: 100% similar: flows/feature.yaml: state "verify": "Check the branch as your role describes. On a later visit your previous resul..." ~ flows/fix.yaml: state "verify": "Check the branch as your role describes. On a later visit your previous resul..."
- yellow: 100% similar: flows/feature.yaml: state "merge": "Follow `lado-checks`, "Merging a run's branch": the **Found on the way** item..." ~ flows/fix.yaml: state "merge": "Follow `lado-checks`, "Merging a run's branch": the **Found on the way** item..."
- yellow: 93% similar: flows/feature.yaml: state "review": "Review the run's branch against main, read-only, as your role describes: the ..." ~ flows/fix.yaml: state "review": "Review the run's branch against main, read-only, as your role describes: the ..."
- yellow: 74% similar: flows/feature.yaml: state "implement": "Implement the design (your `design` input, as the human approved it; read it ..." ~ flows/fix.yaml: state "implement": "Implement the task in the run's worktree, test-first, as your role describes...."
- yellow: 62% similar: agents/architect.md: "Most reviews come as a step of a flow run (a message from `lado`): the design..." ~ agents/reviewer.md: "Most reviews come as a step of a flow run (a message from `lado`): the step s..."

Overall: yellow
```

### Flows

![feature](lado-dev-0.13.0-2026-10-11/feature.svg)

![fix](lado-dev-0.13.0-2026-10-11/fix.svg)

```
$ flow_diagram.py . --compare BLUEPRINT.md
error: BLUEPRINT.md:101: state 'design': a skeleton holds only agent, gate, end, outcomes, max_visits (not produces) (exit status 2)

$ flow_diagram.py . --compare <scratch copy of BLUEPRINT.md without its produces:/reads: lines>
same as the plan in <scratch>/BLUEPRINT-stripped-4.md (exit status 0)
```

## Verdict

**approved.**

- `lado kits check` reports no error.
- No budget measure is red, and every yellow one has its reason in §4.
- The flows agree with the skeletons.
- No high finding is open.

The stop rule is not met: 5 medium findings remain in the changed text. All of them are in
one place: the path for a live run the human asks for at `merge_ok` and a red result sent
back to `implement`.
- Rounds 2–4 added this path, and each round's new sentences opened new edges: 10 medium in round 1, 4 in round 2, 5 now.
- The release, which the brief is about, now has only one low finding left (F3.5).
- This is advice for the human at `release_ok` (question 1).

## Fix first

1. The `implement` `do`s and the developer's own done condition have no exception for an AC marked "needs a live run" (F5.1).
2. The reviewer accepts "needs a live run" on any AC, with no check of when the developer may use it (F4.1).
3. A live run on request has no flaky or environment rule before its red result goes back to `implement` (F3.4).
4. An AC that only a live run can show, on the first `implement` visit, has no path (F3.1, still open).
5. A fix with no reproduction clashes with `diagnosing-bugs` (F5.2).

## Findings

### 1. Role boundaries

No finding. "only the supervisor runs them, locally; no worker does (collecting them with `--collect-only` is not running them)" (`skills/lado-checks/SKILL.md:38-39`).

### 2. Handoffs between steps

No finding. `live-tests` is named both where the supervisor writes it (`lado-checks:48`, `:51-52`) and where the developer reads it ("(its output is the run's `live-tests` artifact)", `agents/developer.md:50`). LADO lets a worker read any artifact of its run by name (pass B: `flows.py:162` rules out a `reads` of an artifact no state produces).

### 3. Done and outcomes

- **F3.1** [medium] STILL OPEN from visit 2; `agents/developer.md:49` (changed text)
  > ``"written, not run", so the human can ask for a run at `merge_ok`. After a rejection for a``
  The "needs a live run" path opens only after a rejection. Two cases still have no path:
  - On the first `implement` visit, an AC that only a live run can show. The developer's live check "is no evidence for an AC" (`:48`), and the done condition cannot hold.
  - A `fix` run started for a release's red live test. It has no `live-tests` artifact; release step 5 names none.
  The developer will report BLOCKED, run a paid live test itself, or claim the AC.
  Fix: open the path to "an AC or a failure that shows only in a live run" (a rejection with `live-tests`, a release's red live test whose output the `fix` task carries, or a provider behaviour). In release step 5, say the live output goes into the `fix` run's task. Passes: 2/3 (A as "partly open", B)

- **F3.4** [medium] `skills/lado-checks/SKILL.md:49` (changed text; known hole 1)
  > `human the command and its last summary line, or the first failing line. A red result is`
  A live run the human asks for during a run goes to the human with no rerun and no class. Step 5 of the release reruns by the flaky rule and classifies; this path does neither. A flaky or environment failure (a provider CLI not logged in, a timeout) can then reach `implement` as a rejection. The developer is told to "reproduce the failure" and changes code for nothing, or marks the AC, and the human pays for another Claude run.
  Fix: after writing `<run>/live-tests`, "rerun a failure as 'When a check fails' says for flaky, each Claude rerun with the human's yes, and tell the human its class; only code or test is a reason to reject the merge." Passes: full pass, confirmed (nothing in `:44-52` points to the flaky rule at `:122`)

- **F3.5** [low] `skills/lado-checks/SKILL.md:82` (changed text)
  > ``--failed`); green on the rerun, go on. Red every time, or live tests red after step 5's``
  CI or the probe going green on a rerun is a flaky test, but this path classifies only "Red every time". So no BACKLOG.md entry is made, though `:87` expects one: "BACKLOG.md entry found during a release (a flaky test) is committed on main after the tag" (R7).
  Fix: "green on the rerun: a flaky test; record it as below, and go on." Passes: 1/3, confirmed (both lines as quoted)

### 4. Independent verification

- **F4.1** [medium] `agents/reviewer.md:41` (changed text; known hole 4)
  > `or command output). An AC the developer reports as "needs a live run" is no finding`
  The developer may use the mark only "After a rejection for a red live run" and only when no non-live layer can catch the failure (`agents/developer.md:49-51`). The reviewer accepts it on any AC, with no check of either condition. It also gets no met / partly / not met mark, though `:40` asks for one on each AC. The pointer "(`lado-checks`, live tests)" leads to a section that never names the mark. An AC can thus leave the independent review untested, and the only remaining check is the human noticing it at `merge_ok`.
  Fix: "An AC marked 'needs a live run' after a rejection for a red live run is 'partly met' and no finding when no non-live test could catch it; when one could, it is an Important Standards finding." Passes: 2/3 (A, B; also the full pass)

### 5. Contradictions

- **F5.1** [medium] `agents/developer.md:52` (changed text; against untouched `flows/feature.yaml:84`, `flows/fix.yaml:20` and `agents/developer.md:41`)
  > `the AC as "needs a live run": that does not block done, and the human asks for the run at`
  Both `implement` `do`s still say "      Done when every AC about behaviour is covered by a test you saw fail and then pass,", and so does the role at `:41`, with no exception. The role gives the step that authority: "the step says what to do and when it is done;" (`:16`). After a red live run that nothing below the live layer reproduces, one developer reports `done` and another reports BLOCKED. `:47` also still calls the written live check "the one exception" to "seen failing", though there are now two.
  Fix: in both `implement` `do`s and at `developer.md:41`: "…saw fail and then pass, except an AC your report marks 'needs a live run' (your role, section 2)"; "the one exception" → "an exception". Passes: 3/3 (also the full pass)

- **F5.2** [medium] `agents/developer.md:51` (changed text; against the dependency skill `diagnosing-bugs`)
  > `lowest non-live layer that can catch it; if none can, report the fix as "not run live" and`
  The developer must "Follow `diagnosing-bugs`: a fast, repeatable reproduction first" (`:65`). That skill says that without a loop it should stop, ask the user for access, and "Do **not** proceed to hypothesise without a loop." (`diagnosing-bugs/SKILL.md:55`). The new text has the developer write a fix with no reproduction. It will either stop and ask, through the supervisor, or write a blind fix.
  Fix: "here a fix without a reproduction loop is allowed: base it on the `live-tests` output and say in your report which hypothesis it rests on." Passes: full pass, confirmed (both quotes)

### 6. Duplication

No finding. The reviewer points to the developer's mark in one line, and the budget script lists only the five pairs already justified in §4.

### 7. When to call the human

No finding. A request that comes too early: "asked earlier, tell the human the run waits for `verify`" (`lado-checks:46-47`). A red release: "Classify the failure ("When a check fails") and tell the human" (`:83`). An environment failure: "they settle it and you rerun" (`:84`).

### 8. Loops on a later visit

No finding. No new loop. A second live run writes a new version of `<run>/live-tests`, and `merge_ok` bounds the `implement` loop.

### 9. Concision and why

No finding. The old F9.1 is fixed. New rules carry their reasons ("so main does not move past the release commit").

### 10. Skill descriptions

No finding. Nothing about skills or `expects` changed. `uv`, needed for `--collect-only`, is in `expects.commands`.

### 11. Provider neutrality

No finding. "Claude" appears only as a `PROVIDER` of LADO's tests.

### 12. Safety and scope

No finding. Each Claude rerun at a release needs a yes ("each Claude rerun with the human's yes", `lado-checks:73-74`). A live run on request waits for `verify` and uses the run's worktree (`:45-47`). The push and the tag each need a yes.

## Known holes

| Known hole | Finding, or how the kit handles it |
|---|---|
| 1. Red check sent back with no environment cause considered | Release: "Classify the failure ("When a check fails") and tell the human" (`lado-checks:83`). A live run during a run: **F3.4**. `verify`: unchanged (`checker.md:53`). |
| 2. Work outside a flow, merge without a gate | "Every task comes as a step of a flow run" (`developer.md:16`); "Any code goes through `fix` or `feature`" (`supervisor.md:72`); merges only after `merge_ok`. |
| 3. Path outside the run's worktree | "Run them in the run's worktree (`flow_status`) at the commit `verify` checked." (`lado-checks:47`), only after `verify` (`:45`). |
| 4. Verdict without a severity threshold | Unchanged: "The answer is Yes when no Critical or Important finding is open; Minor ones do not block". The "needs a live run" AC has no mark: **F4.1**. |
| 5. Dependency skill that writes or asks where its role must not | "a question for the human goes to the supervisor" (`developer.md`, Working rules). `diagnosing-bugs` "Ask the user" clashes with the new path that has no reproduction: **F5.2**. |
| 6. Commands out of step with `expects.commands` | Unchanged: (a) `node` and `npm` are declared; `gh` is the human's choice, guarded by `gh auth status`. (b) The list assumes the LADO repository, by the human's choice. (c) `">=0.34"`; no warning. |
| 7. Read-only roles out of step with what they do | Unchanged: the reviewer is `read_only: true`, its new text makes it write nothing, and `make lint` only checks. "so they do not run on Codex CLI" (`README.md:20`). |

## Not traced

- [low] R8 STILL OPEN.
  - `BLUEPRINT.md:35-36`: "No role" / "  runs live tests in a run: when the change touches a path live tests cover, the checker's".
  - R9 (`:59-61`) and `lado-checks:45-47` have the supervisor run them during a run, at `merge_ok`.
  - The §1 header (`:13`) still says "(live tests only at a release)".
  - Fix: "No worker runs live tests in a run; the supervisor runs them at `merge_ok` when the human asks (R9)."
- [low] R9 has drifted from round 4's order.
  - `BLUEPRINT.md:55`: "  probe or live result is classified first: environment or flaky is rerun (CI or the".
  - The kit reruns CI or the probe once first and classifies only when the rerun is red; the human settles an environment cause before the rerun (`lado-checks:81-84`).
  - Fix: "a red CI or probe is rerun once, live tests by the flaky rule (each Claude rerun with a yes); a green rerun goes on; red every time is classified: environment is settled by the human and rerun, code or test goes through `fix`."
- Otherwise:
  - Every role, step, gate and skill is in §3.
  - Each yellow measure has its reason in §4.
  - The flows match the skeletons ("Flows").

## Previous findings

Findings of this report at `166c46a` (visit 2):

| Finding | Status | Evidence |
|---|---|---|
| F2.1 developer never sees `live-tests` | RESOLVED | `developer.md:50` "(its output is the run's `live-tests` artifact)"; `lado-checks:51-52` "where the developer reads the output in `live-tests`" |
| F3.1 fix or AC that only a live run shows | STILL OPEN (first visit and the release's `fix` run) | The rejection case is fixed: `developer.md:52` 'the AC as "needs a live run": that does not block done'. See F3.1 above. |
| F3.2 CHANGELOG rule never reaches the developer | RESOLVED | `lado-checks:85` "goes through `fix`, whose task says: its CHANGELOG.md line goes under the dated" |
| F3.3 Done when without BACKLOG.md | RESOLVED | `lado-checks:90` "Done when the tag is pushed and each BACKLOG.md entry found during the release is" |
| F5.1 rerun count at a release | RESOLVED | `lado-checks:73-74` "each Claude rerun with the human's yes; green on a rerun, go on."; `:81-82` CI or probe once |
| F5.2 `--collect-only` against "no worker runs them" | RESOLVED | `lado-checks:38-39` "(collecting them with `--collect-only` is not running them)" |
| F9.1 §4 reason for the reviewer | RESOLVED | `BLUEPRINT.md:239` "The reviewer (857) no longer holds a command list (0.13.0); it is yellow for R12's artifact text and its UI review (R10)." |
| F12.1 live run asked for before `verify` | RESOLVED | `lado-checks:45-47` "only once `verify` is green (at `merge_ok`, … asked earlier, tell the human the run waits for `verify`." |
| Not traced: R8/R9 | R9 RESOLVED, R8 STILL OPEN | R9 at `BLUEPRINT.md:51-63`; R8 at `:35-36` unchanged (see "Not traced") |

## Cut rules

| Removed rule (file:line at `166c46a`) | Where it is now |
|---|---|
| "never while a check runs there" (`lado-checks:46`) | "only once `verify` is green" (`:45`), and "Never start a second long check while one runs" (`:57`) |
| Classify first; environment or flaky rerun once; code or test via `fix` (`lado-checks:78-82`) | `lado-checks:81-86`, rerun first, then classify; F3.5 for the green rerun |
| "Done when the tag is pushed, or the human chose to stop …" (`lado-checks:86`) | `lado-checks:90-91`, with BACKLOG.md |
| R9 "then the supervisor runs them locally, with the same yes for Claude" (`BLUEPRINT.md:53`) | `BLUEPRINT.md:59-61` |

No rule lost.

## Missed earlier

None.

## Left by the plan

None: the plan leaves no finding.

## Questions for the human

1. The stop rule is not met. Release 0.13.0 now, or run a fifth build round? I recommend releasing now and taking F3.1, F3.4, F4.1, F5.1 and F5.2 into 0.13.1 together.
   - All five sit on one rare path: you ask for live tests at `merge_ok`, and they come back red.
   - Each one ends at your `merge_ok`, so nothing merges or tags without you.
   - Rounds 2–4 each fixed the last report's findings, then found new edges in their own new sentences. Another round on this path is likely to do the same.
   - For 0.13.1, consider a simpler shape instead of more rules. For example: a red live run asked for at `merge_ok` is the human's to settle (reject with the line, and the developer treats it as any failing test it can reproduce, or BLOCKED), with no "needs a live run" mark for the reviewer to weigh.
2. BLUEPRINT.md R8 and R9 ("Not traced") are the supervisor's text: restate them before the tag? I recommend yes; it is two sentences and changes no behaviour.

## Found on the way

- [kit-builder] `kit-budget`'s `flow_diagram.py --compare` still rejects `produces` and `reads` in a blueprint skeleton (reported in the 0.12.0 report).
- LADO's AGENTS.md:1446-1447 still says live tests run "and a provider's after changing it". The kit overrides it (`lado-checks:39-41`), and the brief's "After the tag" drops it in LADO.
