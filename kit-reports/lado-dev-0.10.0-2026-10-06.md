# Kit report: lado-dev 0.10.0

- Дата: 2026-10-06
- Кит: worktree run'а `improve/lado-dev` (`.lado/worktrees/kit-lado-dev/improve-lado-dev`), путь из задачи шага; ветка `lado/kit-lado-dev/improve-lado-dev`
- Коммит: `a466a55`
- Оценивал: критик kit-builder (слои a и b)
- Режим: повторная оценка, визит 3 шага `evaluate` в `improve`, нужен вердикт.
  - База (`kit-reports/lado-dev-0.9.4-2026-10-06.md`, коммит `58ea55f`) — коммит, который называет план; она одна на всех визитах.
  - Изменение: `git diff 58ea55f -- kit.yaml README.md BLUEPRINT.md agents flows skills`.
  - Находки прошлого визита — из версии этого отчёта в `485fab5`.
- Проходы: три независимых суб-агента прошли по изменённому тексту (весь diff от `58ea55f`), особенно по последнему раунду (`git diff 485fab5`).
  - Проход 1 начинал с ролей.
  - Проход 2 начинал с flows и проследил три сценария: отказ человека от платного прогона, `red` из merge, релиз.
  - Проход 3 начинал с `lado-checks` и применил процедуру к шести реальным путям LADO (`f1ab799`), запуская grep без тестов.
  - Проверку вырезанных правил делали критик и проход 3. Потерянных правил нет.
  - Полный проход по всем 10 файлам, которые затронул diff (Re-evaluation 5), делался параллельно, четвёртым суб-агентом, вместе со всеми зависимыми скиллами. Критик сам подтвердил его находки в файлах и в репозитории LADO.
  - Отброшена 1 однопроходная находка (low): изменённый `conftest.py`/хелпер по строке «A test file» запускает ноль тестов.
  - Как подтверждённые оставлены 2 однопроходные находки: F3.3 и F3.4.

Находки — кандидаты, которые взвешивает человек. Это не оценка «прошёл / не прошёл».

## Card

| Слой | Результат |
|---|---|
| a. `lado kits check` | OK, 0 предупреждений |
| a. Бюджет | жёлтый. Все числовые меры зелёные; жёлтые только 3 пары похожих абзацев (92%, 79%, 77%) — их оставил человек, причина в BLUEPRINT.md §4. Слов в ролях: developer 798, supervisor 798, reviewer 782 при пороге 800. |
| b. Рубрика | 11 находок в изменённом тексте: 0 high, 6 medium, 5 low. Ещё 4 в «Missed earlier» (1 medium, 3 low). Без находок в изменённом тексте 9 из 12 критериев (1, 2, 4, 7, 8, 9, 10, 11, 12); у критериев 2 и 5 есть находки в «Missed earlier». Все 7 находок прошлого визита — RESOLVED. |
| Охват | повторная оценка изменённого текста плюс полный проход по 10 файлам, которые затронул diff (kit.yaml, README.md, BLUEPRINT.md, 4 роли, 2 flow, `lado-checks`) |
| Stop rule | не выполнено: 6 medium в изменённом тексте. Это совет для гейта релиза, не блок. |

Счёт относится только к тому, что указано в строке «Охват»; сравнивать его со счётом полной оценки 0.9.4 нельзя.

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

Вердикт `approved`: блокирующих находок нет. Ниже — что стоит исправить до релиза или следующим раундом. Всё, кроме пункта 5, правится в `skills/lado-checks/SKILL.md`.

1. При «нет» человека на Claude всё равно прогонять бесплатные провайдеры (Kilo, OpenCode) и писать в отчёте «the human declined». Для изменения одного провайдера прогонять только его (F3.1).
2. Убрать неоднозначность выбора тестов. Порог задать жёстко: «more than 10». `make test` заменяет только unit-попадания. `tests/conftest.py` означает только unit-файлы `tests/` (F3.2).
3. Попадание в `tests/ui/conftest.py` через поиск по пакету не должно тащить весь UI для изменения провайдера (F3.3).
4. Пропущенный (skipped) провайдер не считается зелёным live-прогоном (F3.4).
5. Шаг merge: явный исход для flaky — перезапустить `make check` один раз (F3.5). В `agents/reviewer.md:65` добавить одобренные макеты к источникам, которые важнее critique-скиллов (F5.1).

## Findings

### 1. Role boundaries

Права ролей заданы явно: ревьюер и архитектор «You change no files», merge выполняет только супервизор. Скиллы, которые пишут, убраны из ролей, где писать нельзя.

### 2. Handoffs between steps

В изменённом тексте нот без источника нет: `red` передаёт «the failing output in the note», `merge_ok` с `needs: [implement]` показывает отчёт разработчика с concern. Ещё одна находка, в тексте, который diff не трогал, вынесена в «Missed earlier» (F2.1).

### 3. Done and outcomes

- **F3.1** [medium] `skills/lado-checks/SKILL.md:59`
  > no, the developer's report says the live check did not run (DONE_WITH_CONCERNS).

  При «нет» на Claude разработчик не запускает вообще ни одного live-теста. Хотя Kilo и OpenCode работают на бесплатных моделях (`tests/live/conftest.py`, `MODELS`), а правило релиза в `:82` в этом случае велит «run the other providers' live tests». Не сказано, какой `PROVIDER` нужен изменению одного провайдера. Одни агенты спросят «да» на Claude ради правки только Kilo, другие смёрджат изменение hooks/MCP вообще без live-прогона. Кроме того, отчёт пишет «did not run», а исключение ревьюера (`:77`) срабатывает на другое: «unless the report says the human declined the paid run».
  Fix: «On a no, run the other providers' live tests and say in the report that the human declined Claude's (DONE_WITH_CONCERNS). A change to one provider runs that provider's live test.»
  Passes: 3/3

- **F3.2** [medium] `skills/lado-checks/SKILL.md:33`
  > | Anything else: a path no row matches, a module the search finds no test file for or more than about 10 unit test files for | `make test` (all unit tests) |

  Для `src/lado/server/feed.py` поиск (с правилом пакета) находит ровно 11 unit-файлов. Про «about 10» каждый агент решит по-своему. Не сказано, заменяет ли `make test` только unit-попадания или также integration/ui.

  Правило `:44` «stands for every test file in its folder» тоже неоднозначно для попадания `tests/conftest.py` (для `loop`, `tmux`, `terminal`): pytest применяет этот conftest и к подпапкам. Хелпер `tests/agent_helpers.py` используют и integration, и ui.

  Ревьюер сверяет «every row the changed paths need». Если разработчик и ревьюер поняли правило по-разному, появляется находка и лишний круг. Проходы дали low и medium; взят medium.
  Fix: «more than 10»; «`make test` replaces the unit hits only; integration and UI hits still run with their `-m`»; «a hit on `tests/conftest.py` stands for the unit files in `tests/`».
  Passes: 2/3

- **F3.3** [medium] `skills/lado-checks/SKILL.md:37`
  > package, also run it with `P=lado` and `<mod>` set to `<pkg>`: many tests reach a module

  Это правило последнего раунда. Вместе с `:44` (conftest = вся папка) для любого модуля `providers/` поиск находит `tests/ui/conftest.py` и `tests/integration/conftest.py`. Проверено: grep `P=lado`, `<mod>=providers` выдаёт оба файла; в `tests/ui/conftest.py:24` стоит `from lado import providers`. Каждая правка `providers/claude.py` тогда запускает все UI-тесты (после `make web`, `make browser`) и все integration — почти `make check`. Это против R8 и против строки `:28`, которая ограничивает UI экранами для `web/`/`server/`.
  Fix: попадание в `tests/ui/conftest.py` засчитывать, только если путь подходит под строку `web/`/`server/`.
  Passes: 1/3, confirmed (вывод grep в LADO `f1ab799`: `tests/integration/conftest.py … tests/ui/conftest.py`)

- **F3.4** [medium] `skills/lado-checks/SKILL.md:81`
  > pushed commit. A release needs green CI and green live tests on the release commit. If

  Live-тест, у которого CLI не установлен или не залогинен, пропускается: `tests/live/conftest.py:85-88`, `pytest.skip(f"{CLASSES[name].command} not installed")` и `pytest.skip(reason)`. Строка `N passed, M skipped` выглядит зелёной. Релиз или live-проверка разработчика может пройти, ни разу не запустив Claude. Защита в `:48` ловит только «`0 passed` or `no tests ran`».
  Fix: «green means each provider the rule needs passed; a skipped provider is not green: say why (environment, see "When a check fails")».
  Passes: 1/3, confirmed (`tests/live/conftest.py:85-88`)

- **F3.5** [medium] `skills/lado-checks/SKILL.md:103` (state `merge`, обе flow)
  > Report `red`, with the failing output in the note, only for code or test: the task goes

  У шага merge есть исходы для code/test (`red`) и environment (человеку), а для flaky — нет. «When a check fails» (`:119`, «Rerun up to 2 times») не говорит, что перезапускать: упавший тест или весь `make check`. При этом Done требует «the check of step 3 was green on it». Супервизор отправит flaky назад как `red`, или смёрджит после зелёного одиночного теста, или дважды перезапустит 4-минутный `make check` — от прогона к прогону по-разному.
  Fix: в шаге 3: «flaky: rerun the whole `make check` once; green → go on and record the flake in BACKLOG.md on the run's branch; red again → code or test».
  Passes: full pass, confirmed

- **F3.6** [low] `skills/lado-checks/SKILL.md:68`
  > nothing more; after a `red` from the merge step, also the failing tests its note quotes,

  `red` может прийти от `make lint`, `make web` (устаревший `openapi.json`) или `make test-js`, а не только от pytest. Если читать буквально, разработчик перезапускает только «tests».
  Fix: «also the failing checks its note quotes (a test with its `-m`, or the make target)».
  Passes: 2/3

### 4. Independent verification

Ревью, затем `merge_ok`, затем полный `make check` в merge. После `red` run снова проходит `review` и `merge_ok`.

### 5. Contradictions

- **F5.1** [medium] `agents/reviewer.md:83`
  > when no Critical or Important finding is open; Minor ones do not block (a `critique-*`

  Из-за этого правила `major issue` любого `critique-*`-скилла блокирует merge. При этом `:65` («Where a skill disagrees with `docs/design/ui.md` or AGENTS.md, those win») не включает одобренные макеты, а разработчик обязан «Build to the approved mockups … do not redesign» (`agents/developer.md:46-47`). Экран, сделанный точно по одобренному человеком макету, может снова и снова получать `changes` до `max_visits`.
  Fix: `:65` → «Where a skill disagrees with `docs/design/ui.md`, the approved mockups or AGENTS.md, those win». Слов в роли от этого не прибавится больше чем на три.
  Passes: full pass, confirmed

- **F5.2** [low] `agents/supervisor.md:71`
  > the tag only when they are green on the exact release commit.

  `lado-checks:82-83` разрешает релиз по решению человека после отказа от Claude («the human decides whether to release»), а роль запрещает тег без зелёных проверок.
  Fix: в `lado-checks`: «…or the human, having declined Claude's run, decided to release» (роль и так ссылается на скилл).
  Passes: 2/3

- **F5.3** [low] `skills/lado-checks/SKILL.md:50`
  > layer as its own command, and add `-n auto` to every pytest command here: the Makefile

  Сразу за этим идёт абзац о live-тестах (`-m live`). А в Makefile `test-live: … -m live -n0`, и live-тесты идут последовательно: тест Claude использует один и тот же путь к репозиторию.
  Fix: «every unit, integration and UI pytest command (live tests run serially, through `make test-live`)».
  Passes: 2/3

- **F5.4** [low] `skills/lado-checks/SKILL.md:116`
  > - **code**: the product does the wrong thing. Fix the code.

  На «When a check fails» теперь ссылаются ревьюер (`agents/reviewer.md:85-86`, `lado-checks:75`) и шаг merge (`:102`). Но раздел написан для автора: «Fix the code», «record it in BACKLOG.md», а Done — «a fix with the check green again». Ревьюер только читает, супервизор не пишет код, так что ни тот, ни другой этот Done выполнить не может.
  Fix: строка под заголовком: «Only the developer fixes; a reviewer reports the class, the merge step acts as its step 3 says».
  Passes: full pass, confirmed

### 6. Duplication

- **F6.1** [low] `BLUEPRINT.md:33`
  > - **R9** A release needs green CI on the release commit and the live tests on main; no

  Последний раунд изменил правило релиза: теперь «green live tests on the release commit» и путь при отказе (`lado-checks:81-83`). R9 и журнал изменений остались со старым правилом. Следующий `improve`, который будет сверять R9, может вернуть старое правило.
  Fix: обновить R9 и добавить строку в журнал изменений.
  Passes: 3/3

### 7. When to call the human

Платный прогон: «gets the human's yes first, every time, also inside a run» (`lado-checks:57-58`). Отказ человека — «a concern for `merge_ok`, not a finding» (`:77-78`). Сбой окружения в merge: «tell the human what is missing and leave the step open».

### 8. Loops on a later visit

Петля при отказе человека закрыта (`:77`). `red: implement` ограничена `max_visits: 3` у `review` и гейтом `merge_ok`. `implement` и `review` указывают себя в `needs`.

### 9. Concision and why

Нарушений не найдено. Новые правила несут причину: «many tests reach a module through its package», «without it pytest runs one test at a time (minutes instead of seconds)». В ролях 798/798 слов. Строка `:55` и шаг 3 merge (`:101`) в скилле длиннее 100 символов — это оформление, не находка.

### 10. Skill descriptions

Описание `lado-checks` говорит, что в скилле и когда его брать, и отделяет его от `verification-before-completion` и `diagnosing-bugs`. Каждая объявленная папка используется ролью.

### 11. Provider neutrality

Имён CLI-инструментов нет. `PROVIDER` — переменная Makefile LADO. Grep скилла работает в BSD grep и в ugrep.

### 12. Safety and scope

Merge — после `merge_ok`. Платный прогон — только с «да» человека, в том числе при релизе (`agents/supervisor.md:80-81`). Релиз — только по просьбе человека.

## Known holes

| Known hole | Finding, or how the kit handles it |
|---|---|
| 1. Red check sent back with no environment cause considered | Закрыто. Merge: «only for code or test … For environment, tell the human what is missing and leave the step open» (`skills/lado-checks/SKILL.md:103-105`). Ревью: «a check that cannot run for an environment cause is no finding» (`:74-75`), `agents/reviewer.md:86`. Открыт только исход flaky в merge: **F3.5**. |
| 2. Work outside a flow, merge without a gate | Закрыто: «Any code goes through `fix` or `feature`, so every merge into main has a review and the human's `merge_ok`» (`agents/supervisor.md:61-63`). Остаток в тексте, который diff не трогал: **F5.6**. |
| 3. Path outside the run's worktree | Закрыто. Бриф и макеты — абсолютные пути в checkout супервизора (их исключает `.git/info/exclude` LADO). Лог — «outside the tree». |
| 4. Verdict without a severity threshold | Закрыто: «The answer is Yes when no Critical or Important finding is open» (`agents/reviewer.md:82-83`). Связка critique↔макеты — **F5.1**. |
| 5. Dependency skill that writes or asks where its role must not | Закрыто: `tdd` — «without asking the human» (`agents/developer.md:35`); `frontend-design` — «do not redesign or ask the human for a look»; `brainstorming` и `domain-modeling` убраны. |

## Not traced

Нет. Каждая роль, шаг, гейт и скилл есть в трассировке BLUEPRINT.md §3; жёлтая мера обоснована в §4. Устаревший R9 — F6.1.

## Previous findings

Находки прошлого визита (`485fab5`):

| Находка | Статус | Доказательство |
|---|---|---|
| F3.1 отсутствие live при «нет» — Important, петля | RESOLVED | `:76-78` «unless the report says the human declined the paid run: then it is a concern for `merge_ok`, not a finding». Остаток по формулировке отчёта — в F3.1 этого визита. |
| F3.2 grep пропускает тесты, идущие через пакет | RESOLVED | `:36-38` «For a module in a package, also run it with `P=lado` and `<mod>` set to `<pkg>`»; grep находит `test_providers.py`. Побочный эффект — F3.3. |
| F3.4 pytest без `-n auto` | RESOLVED | `:50-51` «add `-n auto` to every pytest command here». Пересечение с live — F5.3. |
| F5.1 environment в ревью — Important | RESOLVED | `:73-75` «a check that cannot run for an environment cause is no finding» |
| F3.3 изменённый `tests/live/` не запускается | RESOLVED | `:31` «\| `tests/live/` \| the live tests, as below \|»; триггер `:55` «or `tests/live/`» |
| F5.2 `make check` в merge для репозитория кита | RESOLVED | `:101` «Run `make check` (in a kit's own repository, `lado kits check .`)» |
| F5.3 релиз: main или коммит релиза, «нет» на Claude | RESOLVED | `:81-83` «green live tests on the release commit. If the human declines Claude's paid run, run the other providers' live tests, and the human decides whether to release». Расхождение с ролью — F5.2. |

Находки первого визита (9) и отчёта 0.9.4 (11) по-прежнему RESOLVED: их места последний раунд не трогал, кроме уже отмеченных выше.

## Cut rules

| Removed rule (file:line at base) | Where it is now |
|---|---|
| Правила, перечисленные на визитах 1–2 | без изменений, см. версии отчёта в `c615612` и `485fab5` |
| «Important when your own run of it is red or cannot run» (`485fab5:SKILL.md:69`) | заменено намеренно (F5.1 прошлого визита): `:73-75` |
| «A release needs green CI on the release commit and the live tests on main» (`485fab5:SKILL.md:75-76`) | `:81-83`, «on the release commit» — намеренно (F5.3 прошлого визита). В BLUEPRINT R9 не перенесено — F6.1. |
| «`make check` was green on it» (Done шага merge) | `:107` «the check of step 3 was green on it» |

Потерянных правил нет.

## Missed earlier

Находки полного прохода в тексте, который diff не трогал. Вердикт они не блокируют и в stop rule не учитываются.

- **F5.5** [medium] `skills/lado-checks/SKILL.md:144`
  > Keep it to a few lines; check first that no entry covers it already. Add it at the end of

  У BACKLOG.md в LADO есть своё правило (BACKLOG.md:5-7): «Entries sit in four tiers … a new entry goes into its tier with a `Size:` and `Why here:` line under its title». Последний уровень — `# P3: maybe never` (строка 1041). Каждая запись, добавленная по киту «at the end», попадает в P3 и без `Size:`/`Why here:`.
  Fix: «Add it at the end of the tier it belongs to (P0–P3, as BACKLOG.md's header says), with its `Size:` and `Why here:` line».
  Passes: full pass, confirmed (`grep -n "^# " BACKLOG.md` → 9/100/506/1041)

- **F5.6** [low] `agents/developer.md:64`
  > nor left unsaid: add a BACKLOG.md entry on your branch (`lado-checks`), and one for each

  Разработчик, запущенный вне run'а, коммитит запись на свою ветку. А `lado-checks:155` говорит: «Outside a run, the supervisor adds them on main». `finish_worker` отказывается, пока ветка не слита («It refuses while the branch is not merged», `mcp_server.py`), и супервизор «fix its reason first», то есть сливает ветку без ревью (known hole 2). Слито будет только BACKLOG.md, поэтому low.
  Fix: в §4 разработчика: «outside a run, list them under **Found on the way**; the supervisor records them».
  Passes: full pass, confirmed

- **F5.7** [low] `skills/lado-checks/SKILL.md:154`
  > **Found on the way** item of the run that has no entry on main yet, since the run's branch

  Строка продолжается «is removed with it». Но `flow_cancel` оставляет worktree и ветку: «its worktree and branch are kept for you to merge or remove» (`mcp_server.py`, `flow_cancel`). То же предположение в `agents/supervisor.md:32`. Брошенные ветки копятся.
  Fix: «…since the run's branch is not merged; tell the human its worktree and branch are kept, and remove them on their yes».
  Passes: full pass, confirmed

- **F2.1** [low] `agents/supervisor.md:51`
  > A UI design starts from `docs/design/ui.md` and updates it. Show the human mockups:

  Не сказано, кто и на какой ветке правит `ui.md`. Если правка лежит в checkout супервизора на main, она мешает `git merge --ff-only` шага merge. Если она закоммичена на main, это коммит без ревью.
  Fix: «…and the design says what to change in it; the developer makes that change on the run's branch».
  Passes: full pass, confirmed

## Left by the plan

- Похожие абзацы (бюджет жёлтый: review 92%, implement 79%, вступления architect/reviewer 77%). Причина человека: две flow задуманы так (R2, R6), шаги различаются источником AC, каждый `do` читается сам. Записано в BLUEPRINT.md §4.

## Found on the way

- `[lado]` `tests/integration/test_server_process.py::test_ui_says_at_once_when_the_server_it_started_exits` упал один раз при параллельном прогоне integration на прошлом визите; вероятно, flaky или окружение.
- `[lado]` Описание `flow_cancel` («its worktree and branch are kept») и текст кита расходятся (F5.7). Стоит решить, на чьей стороне правда: в инструменте или в ките.
