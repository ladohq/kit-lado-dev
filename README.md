# kit-lado-dev

The `lado-dev` kit for [LADO](https://github.com/ladohq/lado) (Layered Agent Delegation &
Orchestration): develop LADO itself for the long term. A supervisor designs with the human
and delegates, an architect reviews each design against the roadmap, developers work
test-first, and reviewers check each branch before it is merged. The flows `feature` and
`fix` take each task from design to merge.

## Use

```bash
lado kits add lado-dev -m official                          # from the official marketplace
lado kits add https://github.com/ladohq/kit-lado-dev.git    # or straight from git (latest vX.Y.Z)
lado start <repo> --kit lado-dev
```

Installing by address or from a marketplace needs LADO 0.20 or later; the kit itself
runs with LADO 0.19 or later (`dependencies.lado` in `kit.yaml`).

## Skill packs

`lado-dev` uses skills from these packs by name and does not copy them; `kit.yaml` pins
each one (`dependencies.skills`). `anthropic-plugins` (the frontend-design plugin) is
Apache-2.0 licensed, the others MIT; thanks to their authors.

| Name | Source | Author | Skills used |
|---|---|---|---|
| `superpowers` | https://github.com/obra/superpowers (tag `v6.4.1`) | Jesse Vincent | brainstorming, verification-before-completion, receiving-code-review |
| `mattpocock-skills` | https://github.com/mattpocock/skills (tag `v1.2.3`) | Matt Pocock | grilling, writing-for-agents, tdd, diagnosing-bugs, codebase-design, domain-modeling |
| `anthropic-plugins` | https://github.com/anthropics/claude-plugins-official (commit `ab024cd`) | Anthropic | frontend-design |
| `designer-skills` | https://github.com/Owl-Listener/designer-skills (commit `9a6930c`) | MC Dean | critique-affordance, critique-color, critique-composition, critique-information-density, critique-typography, critique-visual-hierarchy, feedback-patterns, loading-states, error-handling-ux, navigation-patterns, state-machine |

## Releases

`version` in `kit.yaml` follows semver (major: a role, skill or option users rely on is
removed or renamed; minor: new roles, skills or behaviour; patch: wording and fixes).
Each release is a tag `vX.Y.Z` equal to that `version`: change `kit.yaml`, commit, tag,
push the tag. Check before tagging: `lado kits check . --tag vX.Y.Z`.

History before 0.9.1 comes from `kits/lado-dev` of the former lado-kits repository.
