# Kit report: lado-dev 0.9.1

- Date: 2026-10-06
- Kit: /Users/aleksejkolesnikov/IdeaProjects/kit-lado-dev, path given in the task (commit `c8afb8f`)
- Evaluated by: kit-builder critic (layers a and b)
- Passes: three independent sub-agents, each given only the rubric, `lado-kit-format` and the kit folder. Pass 1 started from kit.yaml and the roles, pass 2 from the flows, step by step, and pass 3 from the skills and then the roles. The critic merged their lists and checked every quote with `grep -nF`. It also checked what the two superpowers skills do against the pinned v6.4.1 copy in `~/.lado/cache`. 9 one-pass findings were dropped. One 2/3 finding was set aside (see 6. Duplication).

Findings are candidates for the human to weigh, not a pass/fail grade.

## Card

| Layer | Result |
|---|---|
| a. `lado kits check` | OK; 0 warnings |
| a. Budget | yellow. Yellow measures: `agents/developer.md` has 904 words, `agents/supervisor.md` has 1273 words, and 1 paragraph is duplicated (the `merge` step in feature and fix) |
| b. Rubric | 13 findings (0 high, 9 medium, 4 low); 5 of 12 criteria have no findings (1, 3, 4, 8, 11) |

### `lado kits check /Users/aleksejkolesnikov/IdeaProjects/kit-lado-dev`

```
lado-dev: OK (3 agents, 71 skills, 4 packs and 2 flows)
```

### Budget script (exit status 0)

```
# Complexity budget: lado-dev 0.9.1

| Measure | Where | Value | Green / yellow up to | Zone |
|---|---|---|---|---|
| Worker roles (not supervisor) | kit | 3 | 3 / 5 | green |
| Work steps in a flow | flows/feature.yaml | 5 | 5 / 8 | green |
| Work steps in a flow | flows/fix.yaml | 3 | 5 / 8 | green |
| Gates in a flow | flows/feature.yaml | 2 | 2 / 3 | green |
| Gates in a flow | flows/fix.yaml | 1 | 2 / 3 | green |
| Words in a role prompt | agents/architect.md | 589 | 800 / 1500 | green |
| Words in a role prompt | agents/developer.md | 904 | 800 / 1500 | yellow |
| Words in a role prompt | agents/reviewer.md | 790 | 800 / 1500 | green |
| Words in a role prompt | agents/supervisor.md | 1273 | 800 / 1500 | yellow |
| Own skills | kit | 1 | 5 / 10 | green |
| MCP servers | kit | 0 | 2 / 4 | green |

## Duplicate paragraphs (one rule, one place)

- yellow: "Record what the review found on the way, then check the branch together with the current main bef..." in flows/feature.yaml: state "merge"; flows/fix.yaml: state "merge"

Overall: yellow
```

The kit has no `BLUEPRINT.md`, so nothing in the kit justifies the three yellow measures.

## Fix first

1. Tell the supervisor what to do with a developer's NEEDS_CONTEXT or BLOCKED message. Today a run can stop at `implement` and nobody acts (F7.1).
2. Make the docs-only check rule the same everywhere. Today `lado-checks` says both "`make check` always" and "`make lint` only", and the `implement` step requires `make check` (F5.1).
3. Make the role and the step agree on rebase vs merge on later visits of `implement`. A rebase rewrites the commits the review and the merge step's skip rule refer to (F5.2).
4. Remove `finishing-a-development-branch` from the supervisor and `requesting-code-review` from the reviewer. These skills bring in push/PR and "dispatch a subagent" procedures that compete with the flow (F10.2, F10.1).
5. Require the human's yes for a paid `make test-live` inside a run, not only for a release (F12.1).

## Findings

### 1. Role boundaries

No findings. Each role states its rights. Architect: "You change no files." Reviewer: "You change no files: no fixes, no commits on the branch you review." Supervisor: "You do not write code yourself". The developer works "only inside your worktree". No action has two owners.

### 2. Handoffs between steps

- **F2.1** [medium] `flows/feature.yaml:98` (state `merge_ok`; the same at `flows/fix.yaml:41`)
  > `    ask: Merge?`
  The gate has no `needs`, so the human sees only the review note. The developer's report includes a section addressed to the human ("Concerns: anything the reviewer or the human should look at", `agents/developer.md:96`) and its deviations from the brief. Neither reaches the merge gate unless the reviewer happens to repeat them.
  Fix: add `needs: [implement]` to `merge_ok` in both flows, or tell the reviewer to carry the developer's Concerns and Deviations into its review. Passes: 2/3

### 3. Done and outcomes

No findings. Every work `do` gives a done condition and a condition per outcome. For example, in `merge`: "If it conflicts, `git merge --abort` and report `conflict`" and "If it is red, report `red`". In `architecture`: "Report `approved` when no Critical or Important finding and no question for the human is left, otherwise `changes`". The gap for developer statuses that end no step is covered under F7.1.

### 4. Independent verification

No findings. Both flows go through `review` (another role) and then the human gate `merge_ok` before `merge`. In `feature`, the architect and `design_ok` (`needs: [design]`) check the design before `implement`. The supervisor rule says: "Every branch is reviewed before it is merged, however small; never skip a flow's review." What the merge gate does not show is covered under F2.1.

### 5. Contradictions

- **F5.1** [medium] `skills/lado-checks/SKILL.md:19` against `skills/lado-checks/SKILL.md:31`
  > `| The change is ready | `make check` | Developer: always, last, before you report done. Reviewer and merge step: as the rules below say. It runs all four above. |`
  > `- A change only to non-code paths (defined below) needs `make lint` only; a change to a`
  For a docs-only change, one line says the developer always runs `make check` and the other says `make lint` is enough. `agents/developer.md:80` and both `implement` steps ("Run `make check` after your last change", whose done condition is "`make check` is green") take the first side. A docs `fix` run will get `make check` in one run and only `make lint` in another.
  Fix: decide once. Either change the table cell to "Developer: last, before you report done, except a non-code-only change (below)", or delete the lint-only clause on line 31. Then align the `implement` steps. Passes: 3/3

- **F5.2** [medium] `agents/developer.md:28` against `flows/feature.yaml:68` (state `implement`; the same at `flows/fix.yaml:12`)
  > `3. Rebase your branch on `main` before the first change. Done when `git log` shows your`
  > `      step's note says what to fix: review findings, a conflict with main (merge main into`
  The role says rebase and the step says merge. "Before the first change" can be read as "on every visit". A rebase on a later visit rewrites commits that were already reviewed, including the commit the reviewer pinned its green `make check` to and the merge step's BACKLOG commit. After that, neither skip rule for `make check` can hold.
  Fix: "Rebase on `main` only before your first commit on the branch; after that, merge main as the step says". Or drop the rebase, since a run's worktree starts from `main`. Passes: 3/3

- **F5.3** [low] `agents/developer.md:103` (state `implement`, later visit)
  > `report back as in 5: summary = status and how many findings you fixed and disputed, body =`
  Section 6 says the report body after a review is "each finding with what you did". The step says note_body "starts with every AC, written out in full, with its status" (`flows/feature.yaml:75-76`, `flows/fix.yaml:19-20`). On a later visit the developer may drop the AC list, and the re-reviewer then loses the per-AC status the `review` step refers to.
  Fix: in section 6, "body = the full report as in 5, AC list first, then each finding with what you did or why you dispute it". Passes: 2/3

### 6. Duplication

- **F6.1** [medium] `flows/feature.yaml:110` (state `merge`), `flows/fix.yaml:53`, `skills/lado-checks/SKILL.md:43`
  > `      3. Skip `make check` only when both hold (`lado-checks`, "Run as little as proves`
  Both `merge` steps point to `lado-checks` and then write out its whole skip rule and its never-pipe rule anyway. The budget script also reports the two `merge` paragraphs as duplicates. The copies already differ in wording: "such as your BACKLOG.md commit" vs "such as the merge step's BACKLOG.md commit". The reviewer's re-review rule (`agents/reviewer.md:33-36`) copies `lado-checks:40-42` the same way. Three copies of a gating rule will drift apart.
  Fix: keep the rule in `lado-checks`. In each `merge` step keep "skip `make check` only as `lado-checks`, 'Run as little as proves the claim', allows; say which commit's check stands". Shorten the reviewer's lines the same way. Passes: 3/3

- **F6.2** [low] `agents/supervisor.md:121`
  > `- A bug, friction or debt in LADO that you find goes to BACKLOG.md (`lado-checks` says who`
  Lines 121-125 spell out who records which **Found on the way** items. `lado-checks:93-99` already defines that, and it is restated again in `agents/developer.md:72-76` and in both `implement` steps.
  Fix: cut it to "A bug, friction or debt you find goes to BACKLOG.md as `lado-checks` says". Keep only the merge-step and cancel duties where they are acted on. Passes: 3/3

- **F6.3** [low] `flows/feature.yaml:19` (state `design`)
  > `      3. Write a short design: the root cause (for a fix), 2-3 options with their trade-offs`
  The `design` step repeats what the role says in "Design for the long term" (`agents/supervisor.md:52-63`): root cause, 2–3 options with their long-term cost, fit with ROADMAP. It also repeats that the design must stand alone as the developer's brief (`feature.yaml:34-36` vs `supervisor.md:72-73`). The step points to that section in its first line anyway.
  Fix: keep the content rules in the role. Leave in the step the design's required parts (ACs, **Found on the way**), the later-visit rule, the done condition and the outcome. Passes: 2/3

Set aside: two passes noted that the reviewing roles and their `do`s both say "mark each of its findings RESOLVED or STILL OPEN". `lado-kit-format` asks the `do` of a reviewing state on a loop to say what changes on a later visit, so this restatement is required by the format. It is not listed as a finding.

### 7. When to call the human

- **F7.1** [medium] `agents/developer.md:88` (state `implement`, both flows)
  > `   BLOCKED do not finish a step: send them to the supervisor with `send_message` and leave`
  `implement` has only the outcome `done`. A NEEDS_CONTEXT or BLOCKED report therefore leaves the run parked. `agents/supervisor.md` and the flows never mention these statuses (grep finds nothing). The supervisor has no rule on whether to answer from the brief, take the question to the human, or `flow_cancel`, so a blocked run waits until someone notices. A `make check` that fails for an environment reason ends in the same place ("Report what is missing", `lado-checks`).
  Fix: add one working rule to the supervisor. On a worker's NEEDS_CONTEXT or BLOCKED, answer from the brief or design with `send_message`, or ask the human and pass the answer on. Cancel the run only on the human's decision. Passes: 3/3

### 8. Loops on a later visit

No findings. `design` needs itself and revises "that text, not one from memory". `architecture` (`needs: [design, architecture]`) and `review` (`needs: [design, review]` / `[review]`) mark findings "RESOLVED or STILL OPEN first", and both have `max_visits: 3`. `implement` writes its whole report on every visit. The `merge`-to-`implement` loop goes through the `merge_ok` gate.

### 9. Concision and why

- **F9.1** [low] `skills/lado-checks/SKILL.md:78`
  > ``BACKLOG.md` is the place until the task tracker is connected (ROADMAP stage 9); then the`
  This sentence is roadmap exposition. It changes nothing an agent does today, and it will be wrong once stage 9 lands.
  Fix: delete the sentence. Write it into the skill when the tracker is connected. Passes: 2/3

### 10. Skill descriptions

- **F10.1** [medium] `agents/reviewer.md:6`
  > `  - requesting-code-review`
  Nothing in the reviewer role or its steps calls for this skill, and the skill is written for the author who asks for a review. Its text says "Dispatch a code reviewer subagent" and calls reviewing the diff yourself a mistake ("I'll just review the diff myself instead of dispatching a reviewer"). A reviewer that loads it may hand its review to a sub-agent.
  Fix: remove it from `skills:`. Passes: 3/3

- **F10.2** [medium] `agents/supervisor.md:9`
  > `  - finishing-a-development-branch`
  No line in the supervisor role or the flows calls for this skill. It offers its own menu, which includes "Push and create a Pull Request" (`git push -u origin <feature-branch>`). That competes with the `merge` step's fixed procedure (`git merge main`, `make check`, `git merge --ff-only`), and one of its options pushes without a gate.
  Fix: remove it from `skills:`, or name it in `merge` and say that only its local-merge path applies. Passes: 3/3

### 11. Provider neutrality

No findings. The kit has no CLI tool names, slash commands or model names, and it uses no paths to kit files. Actions are phrased neutrally: "Change files with your editing tools (edit, write)". `PROVIDER=claude` / `kilo` are parameters of LADO's own `make test-live`, the product under development, so they are not a provider dependency of the kit. MCP servers: none.

### 12. Safety and scope

- **F12.1** [medium] `skills/lado-checks/SKILL.md:20`
  > `| A real agent CLI still works | `make test-live PROVIDER=claude` or `PROVIDER=kilo` | After changing a provider, and on main before a release. Ask the supervisor first for Claude: it uses a paid model. |`
  A developer who touched a provider asks the supervisor before a paid run. The supervisor's only paid-run rule is for releases: "main (`lado-checks`; ask the human first for Claude, it uses a paid model)" (`agents/supervisor.md:103`). Inside a run it may therefore approve a paid model run on its own.
  Fix: add to the supervisor's working rules: "a paid `make test-live` needs the human's yes, also inside a run". Or make the skill say "ask the human, through the supervisor". Passes: 3/3

- **F12.2** [medium] `agents/supervisor.md:96`
  > `changes; read the reason and fix that first. Use `discard=True` only for work you decided`
  `finish_worker(..., discard=True)` throws away a worker's unmerged work, and here the supervisor alone decides. Compare `flow_cancel`, which is allowed only for "a run that the human decided to drop" (`supervisor.md:46`).
  Fix: "Use `discard=True` only for work the human decided to throw away." Passes: 2/3

## Questions for the human

1. On a later visit of `implement`, should the developer rebase on `main` or merge it (F5.2)? Recommended: rebase only before the first commit, then always merge. The review's pinned SHA and the merge step's skip rule depend on commits not being rewritten.
2. Does a paid `make test-live PROVIDER=claude` inside a run need your yes every time, or is the supervisor's own decision enough (F12.1)? Recommended: your yes every time, the same rule as for a release.
3. Does a docs-only change need `make check` or only `make lint` (F5.1)? Recommended: `make lint` only, as the "Run as little as proves the claim" section already intends. The table row and the `implement` steps should say the same.
