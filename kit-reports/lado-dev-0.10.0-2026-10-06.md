# Kit report: lado-dev 0.10.0

- Дата: 2026-10-06
- Кит: worktree run'а `improve/lado-dev` (`.lado/worktrees/kit-lado-dev/improve-lado-dev`), путь из задачи шага; ветка `lado/kit-lado-dev/improve-lado-dev`
- Коммит: `467c1df`
- Оценивал: критик kit-builder (слои a и b)
- Режим: повторная оценка, визит 2 шага `evaluate` в `improve` (нужен вердикт).
  - База — коммит, который называет план, на всех визитах: отчёт `kit-reports/lado-dev-0.9.4-2026-10-06.md`, коммит `58ea55f`.
  - Изменение: `git diff 58ea55f -- kit.yaml README.md BLUEPRINT.md agents flows skills`.
  - Находки прошлого визита (версия этого отчёта в `c615612`) отмечены ниже.
- Проходы:
  - Три независимых суб-агента прошли по изменённому тексту: весь diff от `58ea55f`, с упором на последний раунд правок (`git diff c615612`).
    - Проход 1 начинал с ролей и проследил провайдерное изменение, где человек сказал «нет» платному прогону Claude.
    - Проход 2 начинал с flows и проследил implement → review → merge_ok → merge → red → implement.
    - Проход 3 начинал с `lado-checks`: прогнал grep скилла по настоящим модулям LADO (`~/IdeaProjects/lado`, `f1ab799`) и замерил время.
  - Вырезанные правила проверили критик и проход 2.
  - Отброшены 2 однопроходные находки уровня low:
    - какой `PROVIDER` выбирать для изменения одного провайдера;
    - охватывает ли `tests/conftest.py` подпапки и что значит «about 10».
  - Однопроходных находок, оставленных как подтверждённые, нет.
  - Полный проход (Re-evaluation 5) не делался: вердикт `changes`.

Находки — кандидаты, которые взвешивает человек. Это не оценка «прошёл / не прошёл».

## Card

| Слой | Результат |
|---|---|
| a. `lado kits check` | OK, 0 предупреждений |
| a. Бюджет | жёлтый, но все числовые меры зелёные. Жёлтые только три пары похожих абзацев (92%, 79%, 77%); их оставил человек, причина в BLUEPRINT.md §4. Слов в ролях: developer 798, supervisor 798, reviewer 782 при пороге 800. |
| b. Рубрика | 7 находок: 1 high, 6 medium, 0 low. Без находок 10 из 12 критериев (1, 2, 4, 6, 7, 8, 9, 10, 11, 12). Все 9 находок прошлого визита — RESOLVED; 4 новые находки появились из-за правок последнего раунда. |
| Охват | повторная оценка изменённого текста: 9 файлов кита плюс BLUEPRINT.md, без полного прохода (вердикт `changes`) |
| Stop rule | не выполнено: 1 high и 6 medium в изменённом тексте. Это совет для гейта релиза, не блок. |

Счёт относится только к тому, что указано в строке «Охват»; со счётом полной оценки 0.9.4 его сравнивать нельзя.

### `lado kits check .`

```
lado-dev: OK (3 agents, 20 skills, 4 packs and 2 flows)
```

### Budget script (exit status 0)

```
# Complexity budget: lado-dev 0.10.0

| Measure | Where | Value | Green / yellow up to | Zone |
|---|---|---|---|---|
| Worker roles (not supervisor) | kit | 3 | 3 / 5 | green |
| Work steps in a flow | flows/feature.yaml | 5 | 5 / 8 | green |
| Work steps in a flow | flows/fix.yaml | 3 | 5 / 8 | green |
| Gates in a flow | flows/feature.yaml | 2 | 2 / 3 | green |
| Gates in a flow | flows/fix.yaml | 1 | 2 / 3 | green |
| Words in a role prompt | agents/architect.md | 574 | 800 / 1500 | green |
| Words in a role prompt | agents/developer.md | 798 | 800 / 1500 | green |
| Words in a role prompt | agents/reviewer.md | 782 | 800 / 1500 | green |
| Words in the lead's prompt | agents/supervisor.md | 798 | 1000 / 1500 | green |
| Own skills | kit | 1 | 5 / 10 | green |
| MCP servers | kit | 0 | 2 / 4 | green |

## Similar paragraphs (one rule, one place; 55% similar or more)

- yellow: 92% similar: flows/feature.yaml: state "review": "Review the run's branch against main, read-only, as your role describes: the ..." ~ flows/fix.yaml: state "review": "Review the run's branch against main, read-only, as your role describes: the ..."
- yellow: 79% similar: flows/feature.yaml: state "implement": "Implement the design (the note from design, as the human approved it) in the ..." ~ flows/fix.yaml: state "implement": "Implement the task in the run's worktree, test-first, as your role describes...."
- yellow: 77% similar: agents/architect.md: "Most reviews come as a step of a flow run (a message from `lado`): the design..." ~ agents/reviewer.md: "Most reviews come as a step of a flow run (a message from `lado`): the step s..."

Overall: yellow
```

## Fix first

Все пять правок — в `skills/lado-checks/SKILL.md`, роли не удлиняются.

1. Если человек отказал в live-прогоне, его отсутствие не должно быть находкой Important: это вопрос для `merge_ok`. Иначе run крутится до `max_visits` (F3.1).
2. Для модуля пакета искать также импорты самого пакета: на `providers/claude.py` grep сейчас находит только `test_doctor.py` (F3.2).
3. Добавить `-n auto` ко всем pytest-прогонам скилла. Без него целевой набор для `hooks.py` идёт 3 мин 02 с, с ним — 31 с (F3.4).
4. Для ревьюера «cannot run» по причине окружения не должно быть находкой, как и сказано в `agents/reviewer.md:86` (F5.1).
5. Нужна строка таблицы для `tests/live/`: сейчас изменённый live-тест уходит в `make test` и до релиза не запускается (F3.3).

## Findings

### 1. Role boundaries

Изменения последнего раунда не касаются границ ролей. Ревьюер по-прежнему «You change no files». Merge — только у супервизора (`skills/lado-checks/SKILL.md`, «Merging a run's branch»).

### 2. Handoffs between steps

Красный merge передаёт «the failing output in the note» (`skills/lado-checks/SKILL.md:96`). Разработчик берёт его: «after a `red` from the merge step, also the failing tests its note quotes» (`:64`).

### 3. Done and outcomes

- **F3.1** [high] `skills/lado-checks/SKILL.md:72` (state `review`, обе flow)
  > them, or name their absence as an Important finding. Name the commit your checks ran on.

  Если человек говорит «нет» платному прогону, `:55` предписывает: «no, the developer's report says the live check did not run (DONE_WITH_CONCERNS).» А `:72` делает это отсутствие находкой Important, которая блокирует «Yes» (`agents/reviewer.md:82-83`).
  Что происходит дальше:
  - ревьюер отвечает `changes`;
  - разработчик ничего исправить не может, снова спрашивает человека («every time») и снова получает «нет»;
  - цикл идёт до `max_visits: 3` у `review`, затем открывается гейт.

  Отказ человека превращается в петлю ревью. Причина — правка последнего раунда («as an Important finding», исправление F3.3 прошлого визита).
  Fix: «…or name their absence as an Important finding, unless the report says the human declined it: then it is a concern for `merge_ok`, not a finding». Бесплатные провайдеры при этом всё равно прогоняются.
  Passes: 3/3

- **F3.2** [medium] `skills/lado-checks/SKILL.md:26`
  > | `src/lado/**.py` | The test files that import the module (below), each with its layer's `-m` |

  Проверено на LADO `f1ab799`: для `src/lado/providers/claude.py` grep скилла (`P=lado.providers`) находит только `tests/test_doctor.py`. Он пропускает `tests/test_providers.py`, `test_runtime.py`, `test_liveness.py` и `test_server_launch.py`. Эти файлы получают провайдер через `from lado import … providers` и `providers.get("claude")`.
  В итоге разработчик и ревьюер проходят мимо поломки, она всплывает только в merge: лишний круг `red` и второй гейт человека. Старый `grep -rlw` эти файлы находил, так что это регрессия.
  Fix: для `src/lado/<pkg>/<mod>.py` прогонять ещё и файлы, которые grep находит для самого пакета (`P=lado`, `<mod>=<pkg>`).
  Passes: 3/3

- **F3.3** [medium] `skills/lado-checks/SKILL.md:30`
  > | A test file outside `tests/live/` | that file, with its layer's `-m` |

  Роль велит расширять live-сценарий при изменениях «messages, status, worktrees, kits, providers» (`agents/developer.md:40-42`). Триггер live в `:50-51` уже: «a provider, hooks, the MCP server or how agents get their input». Изменённый файл в `tests/live/` не подходит ни под одну строку таблицы и уходит в «Anything else» (`:32`), то есть в `make test`. А `make test` его не запускает, и новая live-проверка впервые пойдёт только на релизе.
  Fix: строка «`tests/live/` | the live tests, under **Live tests** below» и в триггер `:50` добавить «or `tests/live/`».
  Passes: 2/3

- **F3.4** [medium] `skills/lado-checks/SKILL.md:42`
  > in its folder. Run each hit with its folder's layer: `tests/` plain, `tests/integration/`

  `-n auto` задан только в Makefile (`PYTEST_ARGS ?= -n auto`); `addopts` в `pyproject.toml` его не включает. Поэтому все pytest-прогоны скилла идут последовательно: и `uv run pytest -m integration` из `:27`, и прогоны по слоям из `:42-43`.
  Замер прохода 3 на LADO для `hooks.py` (3 unit-файла, 401 тест): 3 мин 02 с последовательно против 31 с с `-n auto`. Это почти как `make test` (3 мин 41 с). Для `state` и `runtime` слои подряд дают по сути `make check` по частям. Это бьёт по R8.
  Fix: одна фраза под таблицей: «Add `-n auto` to every pytest run».
  Passes: 2/3

### 4. Independent verification

После `red` run снова проходит `review` и `merge_ok`. Полный `make check` в merge идёт после гейта.

### 5. Contradictions

- **F5.1** [medium] `skills/lado-checks/SKILL.md:69`
  > check is a finding: Important when your own run of it is red or cannot run (for an

  В роли сказано: «fails", says: an environment failure is no finding; send the supervisor what is missing» (`agents/reviewer.md:86`). Скилл же делает «cannot run» находкой Important, и скобка «(for an environment cause, see …)» читается как часть этого правила. Один ревьюер ответит `changes` и вернёт сбой окружения разработчику (known hole 1), другой оставит шаг открытым. Причина — правка последнего раунда.
  Fix: «Important when your own run of it is red for code or test; when it cannot run for an environment cause, no finding (your role, section 4)».
  Passes: 3/3

- **F5.2** [medium] `skills/lado-checks/SKILL.md:95` (state `merge`)
  > 3. Run `make check`. If it is red, classify the failure as "When a check fails" says.

  `:18` говорит, что в собственном репозитории кита «`lado kits check .` is its only check, during work and at merge». А раздел merge, по которому работает супервизор, и `:73` («`make check`, always») требуют `make check`, которого там нет. Супервизор либо классифицирует это как environment и оставит шаг открытым, либо нарушит «always». Проходы дали low и medium; взят medium.
  Fix: в шаге 3 merge: «Run `make check` (in a kit's own repository, `lado kits check .`)».
  Passes: 2/3

- **F5.3** [medium] `skills/lado-checks/SKILL.md:75`
  > pushed commit. A release needs green CI on the release commit and the live tests on

  Строка заканчивается словом «main», а роль требует: «the tag only when they are green on the exact release commit.» (`agents/supervisor.md:71`). Если после live-прогона появился коммит с номером версии, один супервизор повторит платный прогон и снова спросит человека, другой не станет. Что делать с релизом при «нет» на Claude, не сказано. Проходы дали medium и low; взят medium.
  Fix: «…and the live tests on the release commit; on a no to Claude's, the other providers', and the human decides».
  Passes: 2/3

### 6. Duplication

Строка супервизора о платном прогоне — указатель «(`lado-checks`)». Команд проверки в ролях и flows нет.

### 7. When to call the human

Платный прогон, кто бы его ни запускал: «gets the human's yes first, every time, also inside a run» (`skills/lado-checks/SKILL.md:53-54`). Сбой окружения в merge: «tell the human what is missing and leave the step open».

### 8. Loops on a later visit

Петля `red: implement` ограничена `max_visits: 3` у `review` и гейтом `merge_ok`. Петля при отказе человека — F3.1.

### 9. Concision and why

Роли укладываются в 800 слов. У нового правила есть причина: «Checks are slow (the whole unit suite takes about 4 minutes)». Длинные строки исправлены: в `agents/` и `flows/` нет строк длиннее 95 символов, кроме description во frontmatter.

### 10. Skill descriptions

«partial proves nothing» согласовано с `verification-before-completion` (`skills/lado-checks/SKILL.md:59-61`). Описание скилла говорит, когда его брать.

### 11. Provider neutrality

Имён CLI-инструментов нет. `PROVIDER` — переменная Makefile самого LADO. Grep скилла одинаково работает в ugrep и в macOS `/usr/bin/grep` (проверено).

### 12. Safety and scope

Платный прогон требует «да» человека и при релизе (`agents/supervisor.md:80-81`: «A paid live test, a developer's or yours at a release, needs the human's yes every time»). Строка таблицы исключает `tests/live/` (`:30`). Merge стоит за `merge_ok`.

## Known holes

| Known hole | Finding, or how the kit handles it |
|---|---|
| 1. Red check sent back with no environment cause considered | В merge закрыто: «For environment, tell the human what is missing and leave the step open». В ревью снова открыто правкой скилла: **F5.1**. |
| 2. Work outside a flow, merge without a gate | Закрыто: «Any code goes through `fix` or `feature`, so every merge into main has a review and the human's `merge_ok`» (`agents/supervisor.md:61-63`). |
| 3. Path outside the run's worktree | Новых случаев нет. Лог «outside the tree»; `--ff-only` «In your repo, on main» — только в merge. |
| 4. Verdict without a severity threshold | Закрыто: «The answer is Yes when no Critical or Important finding is open» (`agents/reviewer.md:82-83`); уровни находок о проверках заданы (`skills/lado-checks/SKILL.md:69-72`). Их сочетание с отказом человека даёт **F3.1**. |
| 5. Dependency skill that writes or asks where its role must not | Закрыто: `tdd` перекрыт фразой «without asking the human» (`agents/developer.md:35`); «partial proves nothing» согласовано (`skills/lado-checks/SKILL.md:59-61`). |

## Not traced

Нет. Все элементы есть в таблице трассировки BLUEPRINT.md §3; жёлтая мера обоснована в §4.

## Previous findings

Находки прошлого визита (`c615612`):

| Находка | Статус | Доказательство |
|---|---|---|
| F12.1 строка «A test file» запускает `tests/live` | RESOLVED | `:30` «\| A test file outside `tests/live/` \|»; `:49` «**Live tests** (`-m live`: `make test-live`, anything under `tests/live/`)». Новый побочный эффект — F3.3. |
| F12.2 «да» человека потеряно для релиза | RESOLVED | `:53-54` «whoever runs it, the developer through the supervisor or the supervisor at a release, gets the human's yes first»; `agents/supervisor.md:80` |
| F3.1 grep по словам и неверные имена файлов | RESOLVED | `:38` — grep по импортам. Проверено на LADO: `hooks` → test_liveness, test_providers, test_runtime. Пропуск для модулей пакетов — F3.2. |
| F3.2 «the matching» для integration, «each screen» | RESOLVED | `:27` «those the search below finds, or all of them … when it finds none»; `:28` «by file name; all of `tests/ui/` when unsure» |
| F5.1 «nothing more» против повторного прогона после `red` | RESOLVED | `:64` «after a `red` from the merge step, also the failing tests its note quotes» |
| F3.3 уровень находки «пропущенная проверка» | RESOLVED | `:69-70`. Новые побочные эффекты — F3.1 и F5.1. |
| F3.4 `make lint` в репозитории кита | RESOLVED для работы (`:17-18`) | для merge — F5.2 |
| F5.2 `verification-before-completion` против целевых проверок | RESOLVED | `:59-61` «The rows of the table are the whole proof of a claim while a task is in work» |
| F9.1 длинные строки | RESOLVED | строк длиннее 95 символов в `agents/` и `flows/` нет, кроме description во frontmatter |

Все 11 находок отчёта 0.9.4 остаются RESOLVED: это проверено на первом визите, а последний раунд их мест не трогал.

## Cut rules

| Removed rule (file:line at base) | Where it is now |
|---|---|
| Все правила, перечисленные на первом визите | без изменений, см. версию этого отчёта в `c615612` |
| «(fake agent, no LLM)» у integration (`c615612:SKILL.md:24`) | это пояснение, а не правило; удаление допустимо |
| Строка кита в таблице (`c615612:SKILL.md:29`) | `:17-18`; в merge не перенесено — F5.2 |
| `.md` под `src/` — это код, причина «`make check` validates the built-in kits» (`58ea55f:SKILL.md:40-41`) | правило сохранено (`:31`), потеряна только причина; это low, отдельной находкой не оформлено |

## Missed earlier

Нет: все находки — в тексте, изменённом diff от `58ea55f`.

## Left by the plan

- Похожие абзацы (бюджет жёлтый: review 92%, implement 79%, вступления architect/reviewer 77%). Причина человека: две flow задуманы так (R2, R6), их шаги различаются источником AC, каждый `do` читается сам по себе. Записано в BLUEPRINT.md §4.

## Found on the way

- `[lado]` Проход 3 при параллельном прогоне integration в репозитории LADO получил одно падение: `tests/integration/test_server_process.py::test_ui_says_at_once_when_the_server_it_started_exits`. Вероятно, это flaky-тест или проблема окружения. На кит не влияет, но в LADO его стоит проверить.
