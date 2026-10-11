# Kit report: lado-dev 0.13.0

- Date: 2026-10-11
- Kit: the worktree of run `improve/lado-dev` (`.lado/worktrees/kit-lado-dev/improve-lado-dev`), branch `lado/kit-lado-dev/improve-lado-dev`; path given by the task
- Commit: `e41fcac`
- Evaluated by: kit-builder critic (layers a and b)
- Mode: re-evaluation, visit 2 of step `evaluate` in `improve` (after the human's reject at `release_ok` and build rounds 2 and 3); verdict asked.
  - The previous report is this file at `c5a62a7` (the `evaluation` of visit 1). git keeps that version.
  - The base the plan names is `38324bf`. This visit reviews this round's change, `git diff c5a62a7 -- kit.yaml README.md BLUEPRINT.md agents flows skills`, which touches 6 files: kit.yaml, README.md, BLUEPRINT.md, `agents/developer.md`, `agents/reviewer.md` and `skills/lado-checks/SKILL.md`. It reviews that change in the context of the whole run's change (`git diff 38324bf`).
- Passes:
  - Three independent sub-agents went over this round's diff and its surroundings, each with `kit-rubric`, `lado-kit-format`, the kit, both diffs, the previous report and LADO's repository for facts. Pass A started from kit.yaml and the roles. Pass B started from the flows and traced scenarios: a live run asked for at `merge_ok` and rejected, a release end to end with `gh` not logged in, red CI, a flaky live test and a fix during the release, and a developer adding a live check. Pass C started from `lado-checks` against LADO's repository; it ran `uv run pytest -m live --collect-only tests/live/test_live.py`, which collected 12 tests and called no model.
  - The critic did the cut-rule check.
  - A fourth sub-agent made the full pass (Re-evaluation 5) over the whole of the 6 touched files.
  - The critic checked every quote with `grep -nF`.
  - Dropped: 2 one-pass findings. One said other runs may merge into main during a release; the harm is not shown in the files. The other said `node`/`npm` in `expects` refuse a session in a kit's own repository; that list is the human's choice, as `make`/`uv` were in visit 1.
  - Kept: 3 full-pass findings, as confirmed (F12.1, F3.3, F5.2).

Findings are candidates for the human to weigh, not a pass/fail grade.

## Card

| Layer | Result |
|---|---|
| a. `lado kits check` | OK; 0 warnings; also OK with `--tag v0.13.0` |
| a. Budget | Yellow. Worker roles 4, work steps in `feature` 6, developer 1018 words, reviewer 841 words, 5 pairs of similar paragraphs (100%, 100%, 93%, 74%, 62%). All are justified in BLUEPRINT.md §4; for the reviewer's reason, see F9.1. The lead has 947 words (green). |
| a. Flows | 2 drawn. `--compare BLUEPRINT.md` exits 2 on the skeletons' `produces`/`reads` (kit-builder's script). On a copy without those lines the flows match the skeletons; `produces` and `reads` agree by reading. The flows did not change this round. |
| b. Rubric | 8 findings in the changed text (0 high, 4 medium, 4 low), plus 1 "Not traced" item (low). 7 of 12 criteria without findings (1, 4, 6, 7, 8, 10, 11). All 11 findings of visit 1 are RESOLVED. |
| Covers | Re-evaluation of this round's changed text, with a full pass over the 6 files it touches. |
| Stop rule | Not met: 4 medium findings in the changed text. This is advice for the release gate, not a block. |

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

Read-only: `read-only roles: architect, reviewer`. `dependencies.lado` is `">=0.34"`, which is what `read_only` needs (checked by hand).

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
| Words in a role prompt | agents/developer.md | 1018 | 800 / 1500 | yellow |
| Words in a role prompt | agents/reviewer.md | 841 | 800 / 1500 | yellow |
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
error: BLUEPRINT.md:91: state 'design': a skeleton holds only agent, gate, end, outcomes, max_visits (not produces) (exit status 2)

$ flow_diagram.py . --compare <scratch copy of BLUEPRINT.md without its produces:/reads: lines>
same as the plan in <scratch>/BLUEPRINT-stripped-3.md (exit status 0)
```

## Verdict

**approved.**

- `lado kits check` has no error.
- No budget measure is red, and every yellow measure has its reason in BLUEPRINT.md §4.
  - The reviewer's yellow is the one in doubt. The reviewer was 785 words in 0.10.3. It became yellow (868) when 0.12.0 added R12's artifact text, which §4 names, and it is 841 now that its copy of the developer's list is gone. So R12 still explains it, though §4's wording is loose (F9.1).
- The flows agree with the skeletons.
- No high finding is open.

The stop rule is not met: 4 medium findings remain in the changed text. All of them are about live tests the human asks for during a run (F2.1, F3.1, F12.1) or the rerun count at a release (F5.1). Each fix is a sentence. They are the human's to weigh at `release_ok`.

## Fix first

1. A red live run the human asked for: tell the developer to read `live-tests`, and say what "done" means for a fix it may not run live (F2.1, F3.1).
2. Rerun count at a release: "rerun once" against the flaky rule's "up to 2 times"; say whether a Claude rerun needs a new yes (F5.1).
3. A live run asked for before `verify`: there is no commit "`verify` checked" yet (F12.1).
4. The fix's CHANGELOG rule must reach the developer, through the fix's task (F3.2).
5. BLUEPRINT.md: restate the reviewer's §4 reason, and bring R8 and R9 up to date (F9.1, "Not traced").

## Findings

### 1. Role boundaries

No finding. "only the supervisor runs them, locally; no worker does" (`skills/lado-checks/SKILL.md:38`). The reviewer runs `make lint` only (`agents/reviewer.md:33`).

### 2. Handoffs between steps

- **F2.1** [medium] `skills/lado-checks/SKILL.md:49` (changed text)
  > ``the merge at `merge_ok` quoting that line, which sends it to `implement`.``
  When a live run the human asked for is red and they reject at `merge_ok`, `implement` gets only the line the human quoted. No state produces `live-tests`, so it cannot be in `implement`'s `reads` (`flows/feature.yaml:70` `    reads: [design, architecture-review, review, verify]`). Neither `implement` `do` nor `agents/developer.md` names the artifact: `grep -rn live-tests agents flows` finds nothing. The developer fixes a failure it never sees in full.
  Fix: in both `implement` `do`s (or `developer.md` §2): "after a rejection over a live run, read the run's `live-tests` artifact with `read_artifact`". In this paragraph: "tell the human that the developer reads the output there". Passes: 3/3 (also the full pass)

### 3. Done and outcomes

- **F3.1** [medium] `agents/developer.md:48` (changed text; also `:45` "report why it is not worth it. You write the check but do not run it (live tests run at a")
  > `exception to "seen failing" and is no evidence for an AC: list it in your report as`
  The exception covers a live check the developer writes. It does not cover a fix after a rejection for a red live run, or an AC that only a live run can show. There, the `implement` done condition "every AC about behaviour is covered by a test you saw fail and then pass" still holds, and the developer may not run the test. It will either run `make test-live` itself (a paid run without the human's yes) or report done on a fix nobody saw pass. The reviewer's "met / partly / not met" for such an AC can send the run back up to `max_visits`.
  Fix: "After a rejection for a red live run, reproduce it at the lowest non-live layer that can; if none can, report the fix as 'not run live' and the AC as 'needs a live run': it does not block done, and the human asks for the run at `merge_ok`." In `reviewer.md` §2, the same mark is no finding. Passes: 3/3 (pass A within its F2.1, B, C; also the full pass)

- **F3.2** [low] `skills/lado-checks/SKILL.md:81` (changed text)
  > ``through `fix`; its CHANGELOG.md line goes under the dated `## X.Y.Z`, and it updates the``
  The rule binds the `fix` run's developer, but it sits in "Releasing LADO", the supervisor's section, and nothing has the supervisor pass it on. The developer follows AGENTS.md:205 ("adds a line under `## X.Y.Z (unreleased)`"), a heading step 1 has removed.
  Fix: "…goes through `fix`, whose task says: its CHANGELOG.md line goes under the dated `## X.Y.Z` …". Passes: 3/3

- **F3.3** [low] `skills/lado-checks/SKILL.md:86` (changed text)
  > `Done when the tag is pushed, or the human chose to stop and you told them what is left.`
  The done condition leaves out the BACKLOG.md commit that `:83-84` puts after the tag ("is pushed, so main does not move past the release commit."), so a flaky test found at a release can go unrecorded (R7).
  Fix: "Done when the tag is pushed and each BACKLOG.md entry found during the release is committed on main, or …". Passes: full pass, confirmed

### 4. Independent verification

No finding. `merge_ok` reads `[report, review, verify]`, and the report lists a live check as "written, not run" (`agents/developer.md:48-49`). The tag still comes after CI, the probe, the live tests and the human's yes.

### 5. Contradictions

- **F5.1** [medium] `skills/lado-checks/SKILL.md:79` (changed text; known hole 1)
  > ``for environment or flaky, tell the human and rerun once (a workflow with `gh run rerun``
  Release step 5 already reruns a live failure "as "When a check fails" says for flaky" (`:71`), which is "Rerun only what failed, up to 2 times" (`:117`). This paragraph adds "rerun once" after classifying, though a failure can only be called flaky after a rerun. A red live test therefore gets 1, 2 or 3 reruns, depending on the agent. A Claude rerun is a paid run, and nothing says whether the yes of step 5 covers it ("every time", `:42`). Nothing says that a green rerun goes on to the next step either.
  Fix: "For a workflow, rerun what failed once (`gh run rerun <id> --failed`), telling the human; live tests are rerun only as step 5 says, each Claude rerun with the human's yes. Green on a rerun: go on; red every time: classify and tell the human." Passes: 3/3 (also the full pass)

- **F5.2** [low] `skills/lado-checks/SKILL.md:38` (changed text)
  > `asks for them, and only the supervisor runs them, locally; no worker does. In this kit's`
  `agents/developer.md:46-47` has the developer run `uv run pytest -m live --collect-only <file>`. The skill counts anything with `-m live` as a live test and says no worker runs one. A developer that follows the skill skips the collection check, and one that follows its role seems to break the skill.
  Fix: add "(collecting them with `--collect-only` is not running them)". Passes: full pass, confirmed (pass C ran the collection: 12 tests, no model)

### 6. Duplication

No finding. The reviewer's copy of the developer's list is gone (`agents/reviewer.md:33-34`), and `agents/supervisor.md:89` only points to `lado-checks`. The five pairs the budget script lists are justified in §4.

### 7. When to call the human

No finding. A failed `gh auth status` goes to the human ("if it fails, tell the human", `:62`), and so does a red release (`:78-80`) and a red live run ("A red result is no outcome of yours: the human decides", `:48`).

### 8. Loops on a later visit

No finding. No new loop. A second live run on a later `merge_ok` writes a new version of `<run>/live-tests`.

### 9. Concision and why

- **F9.1** [low] `BLUEPRINT.md:229` (changed text)
  > ``| Words in the longest role prompt | 1018 (developer) | yellow | The human asked for each role's exact commands in its own file (2026-10-07); the developer's list (section 5) is the longest. 0.12.0 adds where each result goes (R12); the reviewer (841) is yellow for the same reason. |``
  The reviewer no longer holds a command list. "for the same reason" therefore reads as the commands reason that no longer applies; only the R12 reading still holds (785 words in 0.10.3, 868 in 0.12.0, 841 now). The build report names it as the human's to restate.
  Fix: "the reviewer (841) is yellow for R12's artifact text and its UI review (R10)". Passes: 3/3 (pass A as "Not traced", C, the full pass)

The new rules carry their reasons ("which calls no model", "so main does not move past the release commit").

### 10. Skill descriptions

No finding. `expects.commands` now covers `make check`'s `node` and `npm` (`kit.yaml:13`, `README.md:21`). `gh` is out by the human's choice and is now guarded by `gh auth status`.

### 11. Provider neutrality

No finding. "Claude" is only a `PROVIDER` of LADO's tests. The Codex CLI note in `README.md:20` is unchanged.

### 12. Safety and scope

- **F12.1** [medium] `skills/lado-checks/SKILL.md:45` (changed text; known hole 3)
  > ``note that no live tests ran): run them in the run's worktree (`flow_status`) at the commit``
  "When the human asks for them during a run" covers every step, but "the commit `verify` checked" exists only after `verify`. Asked during `implement` or `review`, the supervisor has no commit to use. If it checks one out, it does so in the tree the developer is editing, and work can be lost.
  Fix: "Only once `verify` is green (at `merge_ok`); asked earlier, tell the human the run waits for `verify`." Passes: full pass, confirmed

The push and the tag each need the human's yes (`:62-63`, `:75-76`). A paid run needs a yes every time (`:41-42`).

## Known holes

| Known hole | Finding, or how the kit handles it |
|---|---|
| 1. Red check sent back with no environment cause considered | Release: "Red CI, probe or live tests: no tag. Classify the failure first ("When a check fails"):" (`lado-checks:78`). The rerun counts conflict: **F5.1**. In runs, unchanged (`checker.md:53`). |
| 2. Work outside a flow, merge without a gate | "Every task comes as a step of a flow run" (`developer.md:16`); "Any code goes through `fix` or `feature`" (`supervisor.md:72`); each push needs a yes. |
| 3. Path outside the run's worktree | "run them in the run's worktree (`flow_status`)" (`lado-checks:45`); the case before `verify` is **F12.1**. |
| 4. Verdict without a severity threshold | Unchanged: "The answer is Yes when no Critical or Important finding is open; Minor ones do not block" (`reviewer.md`). A live-only AC has no mark: **F3.1**. |
| 5. Dependency skill that writes or asks where its role must not | Unchanged and handled: `developer.md:36-37`, `:118-119`. The designer skills on the read-only reviewer write nothing. |
| 6. Commands out of step with `expects.commands` | (a) `node` and `npm` were added; `gh` is out by the human's choice and guarded by `gh auth status`. (b) `make`, `uv`, `node` and `npm` would refuse a session in a kit's own repository (`lado-checks:16-17`). The human chose this list, so it was dropped as a finding. (c) `">=0.34"`; no warning. |
| 7. Read-only roles out of step with what they do | Unchanged: the architect and reviewer are `read_only: true`, and the reviewer's only command is `make lint`. `lado: ">=0.34"`. "so they do not run on Codex CLI" (`README.md:20`). |

## Not traced

- [low] BLUEPRINT.md R8 and R9 lag behind the kit (changed in this run; passes B, C and the full pass).
  - R8 still says "runs live tests in a run: when the change touches a path live tests cover, the checker's" (`BLUEPRINT.md:36`, after "No role"). But `lado-checks:44-46` has the supervisor run them in the run's worktree when the human asks.
  - R9 says "them: then the supervisor runs them locally, with the same yes for Claude. Red CI or red live tests: no tag; the fix" (`:53`). It names neither the run's worktree, nor classifying a red result first, nor the release commit's tag and BACKLOG.md after it.
  - The 0.13.0 row says only "R8 changed".
  - Fix: R8 "No worker runs live tests in a run; the supervisor runs them in the run's worktree when the human asks"; R9 with the round-2 release rules; "R8, R9 changed" in the row.
- Otherwise: every role, step, gate and skill is in §3. The yellow measures have reasons in §4 (the reviewer's: F9.1). The flows match the skeletons (see "Flows").

## Previous findings

Findings of this report at `c5a62a7` (visit 1); the author's list in `build-report` was checked against the files:

| Finding | Status | Evidence |
|---|---|---|
| F3.1 red at release not classified | RESOLVED | `lado-checks:78` "Red CI, probe or live tests: no tag. Classify the failure first ("When a check fails"):"; `:86` "Done when the tag is pushed, …" (rerun count: new F5.1) |
| F3.2 CHANGELOG during a red release | RESOLVED | `lado-checks:81-82` "its CHANGELOG.md line goes under the dated `## X.Y.Z`, and it updates the date if the day changed." (who tells the developer: new F3.2) |
| F5.1 live check written, not run | RESOLVED | `developer.md:47-49` "A new or changed live check is the one exception to "seen failing" and is no evidence for an AC" (the fix after a red live run: new F3.1) |
| F5.2 flaky at release, tag commit | RESOLVED | `lado-checks:75` "6. Tag the release commit (`git tag vX.Y.Z <the release commit>`) and push the tag, with"; `:83-84` BACKLOG.md after the tag |
| F5.3 AGENTS.md "a provider's after changing it" | RESOLVED | `lado-checks:38-40` "In this kit's runs this replaces AGENTS.md's "and a provider's after changing it" … so a change merged without a live run is no finding."; R8 "Smoke tests are left to CI" (`BLUEPRINT.md:39`) |
| F5.4 live tests on the release commit | RESOLVED | `lado-checks:69` "5. `make test-live` locally, on main at the release commit, for every provider, Claude" |
| F6.1 reviewer's drifting copy | RESOLVED | removed, the human's option 3: `reviewer.md:33-34` "Run `make lint` on the reviewed commit, no other test (the checker runs `make check` in" / "`verify`). Note uncommitted changes in the worker's tree."; `BLUEPRINT.md:191` |
| F9.2 developer "Most tasks" | RESOLVED | `developer.md:16` "Every task comes as a step of a flow run"; `:23` "1. Work only inside the run's worktree." |
| F10.1 `node`, `npm` | RESOLVED | `kit.yaml:13` "  commands: [git, make, uv, node, npm]"; `README.md:21` |
| F10.2 `gh` login | RESOLVED | `lado-checks:62` "2. Check `gh auth status` (step 4 needs it; if it fails, tell the human), then push main" |
| F12.1 live run on request | RESOLVED | `lado-checks:44-49`: the run's worktree at the commit `verify` checked, `<run>/live-tests`, red is the human's call (the run before `verify`: new F12.1; the handoff: new F2.1) |

## Cut rules

| Removed rule (file:line at `c5a62a7`) | Where it is now |
|---|---|
| The reviewer checks the developer's report against its command list; a missing one is Minor (`agents/reviewer.md:34-38`) | Dropped by the human's decision (option 3, 2026-10-11): `make check` in `verify` covers every command; `BLUEPRINT.md:191` |
| "No role runs them in a run." (`lado-checks:38`) | `lado-checks:38` "only the supervisor runs them, locally; no worker does", narrowed on purpose by the plan (F12.1 of visit 1) |
| "Red CI, probe or live tests: no tag; tell the human, quoting the failing line. The fix goes through `fix`, then the release goes on from step 2 with the same version." (`lado-checks:65-66`) | `lado-checks:78-82`, with classification added |
| "6. Push the tag `vX.Y.Z`, with the human's yes; it publishes to PyPI." (`lado-checks:63`) | `lado-checks:75-76` |
| "Push main, with the human's yes. That yes also covers starting the Windows probe in 4." (`lado-checks:53`) | `lado-checks:62-63` |
| Developer "(in a run, the run's worktree)", "Most tasks" (`developer.md:16`, `:23`) | Not needed: every task is a run (old F9.2) |

No rule lost.

## Missed earlier

None.

## Left by the plan

None: the plan leaves no finding.

## Questions for the human

1. The stop rule is not met: should there be a fourth build round for F2.1, F3.1, F5.1 and F12.1, or should 0.13.0 be released now with them taken into 0.13.1? Recommended: release now. None of them can lose a release or merge code unreviewed, since each ends at the human's `merge_ok` or at the release's yes. Each round so far has found new edges in the newest text, which the stop rule calls a sign that more rounds mostly find what is left. F12.1 and F5.1 are one sentence each, if you want them in.
2. BLUEPRINT.md §4 and R8/R9 (F9.1, "Not traced") are the supervisor's text. Should they be restated before the tag? Recommended: yes, it is a few lines and does not change the kit's behaviour.

## Found on the way

- [kit-builder] `kit-budget`'s `flow_diagram.py --compare` still rejects `produces` and `reads` in a blueprint skeleton (reported in the 0.12.0 report).
- LADO's AGENTS.md:1446-1447 still says live tests run "and a provider's after changing it". The kit now overrides it (`lado-checks:38-40`). The brief's "After the tag" plans to drop it in LADO.
