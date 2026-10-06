# Kit report: lado-dev 0.10.0

- Дата: 2026-10-06
- Кит: worktree запуска `improve/lado-dev` (`.lado/worktrees/kit-lado-dev/improve-lado-dev`). Это путь из задачи шага, ветка `lado/kit-lado-dev/improve-lado-dev`.
- Коммит: `0738eee`
- Оценивал: критик kit-builder (слои a и b)
- Режим: повторная оценка относительно `kit-reports/lado-dev-0.9.4-2026-10-06.md`, база `58ea55f` (коммит, названный в плане), команда `git diff 58ea55f -- kit.yaml README.md BLUEPRINT.md agents flows skills`. Шаг `evaluate` в `improve`, нужен вердикт.
- Проходы:
  - Три независимых суб-агента прошли по изменённому тексту и его окружению. Каждый получил рубрику, `lado-kit-format`, папку кита, diff и зависимые скиллы изменённых ролей; отчёты из `kit-reports/` им не давали. Проход 1 начинал с kit.yaml и ролей. Проход 2 шёл по flows и вёл run по цепочке implement → review → merge_ok → merge → red → implement. Проход 3 начинал с `lado-checks` и сверял его с репозиторием LADO.
  - Все три прохода сверяли таблицу `lado-checks` с настоящим репозиторием LADO (`~/IdeaProjects/lado`, коммит `f1ab799`).
  - Вырезанные правила критик проверил сам (`git diff --word-diff 58ea55f`); все три прохода нашли ту же потерю.
  - Однопроходных находок 1, критик её подтвердил (F12.1). Отброшенных нет.
  - Полный проход (Re-evaluation 5) не делался: вердикт `changes`, а он нужен только перед `approved`.

Находки — кандидаты, которые взвешивает человек. Это не оценка «прошёл / не прошёл».

## Card

| Слой | Результат |
|---|---|
| a. `lado kits check` | OK, 0 предупреждений |
| a. Бюджет | жёлтый: все числовые меры зелёные. Жёлтые только три пары похожих абзацев (92%, 79%, 77%); причина записана в BLUEPRINT.md §4, их оставил человек. Слов в ролях: developer 798, supervisor 795, reviewer 782 при пороге 800. |
| b. Рубрика | 9 находок: 1 high, 7 medium, 1 low (одна из medium — потерянное правило). Без находок 8 из 12 критериев (1, 2, 4, 6, 7, 8, 10, 11). Все 11 находок отчёта 0.9.4 — RESOLVED. |
| Охват | повторная оценка изменённого текста: 9 файлов кита + BLUEPRINT.md; полного прохода не было (вердикт `changes`) |
| Stop rule | не выполнено: 1 high и 7 medium в изменённом тексте. Это совет для гейта релиза, не блок. |

Счёт относится только к тому, что указано в строке «Охват». Его нельзя сравнивать со счётом полной оценки 0.9.4.

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
| Words in the lead's prompt | agents/supervisor.md | 795 | 1000 / 1500 | green |
| Own skills | kit | 1 | 5 / 10 | green |
| MCP servers | kit | 0 | 2 / 4 | green |

## Similar paragraphs (one rule, one place; 55% similar or more)

- yellow: 92% similar: flows/feature.yaml: state "review": "Review the run's branch against main, read-only, as your role describes: the ..." ~ flows/fix.yaml: state "review": "Review the run's branch against main, read-only, as your role describes: the ..."
- yellow: 79% similar: flows/feature.yaml: state "implement": "Implement the design (the note from design, as the human approved it) in the ..." ~ flows/fix.yaml: state "implement": "Implement the task in the run's worktree, test-first, as your role describes...."
- yellow: 77% similar: agents/architect.md: "Most reviews come as a step of a flow run (a message from `lado`): the design..." ~ agents/reviewer.md: "Most reviews come as a step of a flow run (a message from `lado`): the step s..."

Overall: yellow
```

## Fix first

Все пять исправлений — правки `skills/lado-checks/SKILL.md`, роли не удлиняются.

1. Строка «A test file» не должна покрывать `tests/live/`. Правило «да» человека должно распространяться на любой прогон `-m live` (F12.1).
2. Вернуть «да» человека для платного live-прогона при релизе: правило должно касаться любого, кто запускает live-тест, а не только разработчика (F12.2).
3. Сделать строку `src/lado/…` исполнимой:
   - grep по импортам, а не по словам;
   - явное соответствие пакетов (`providers/` → `tests/test_providers.py`, `server/` → `tests/test_server*.py`);
   - попадания в integration/ui запускать со своим `-m`;
   - при отсутствии файла или слишком широком совпадении — `make test` (F3.1).

   Так же определить «matching» для integration (F3.2).
4. После `red` из merge разработчик и ревьюер перезапускают и упавшие тесты из ноты merge (F5.1).
5. Дать находке «пропущенная проверка» уровень: Important, если своя проверка ревьюера красная, иначе Minor (F3.3).

## Findings

### 1. Role boundaries

Изменения: `brainstorming` убран у супервизора, `domain-modeling` убран у архитектора (`agents/architect.md:4-6`: «skills: / - lado-checks / - codebase-design»). Ревьюер по-прежнему только читает: «You change no files». Ни одна роль не получила чужого действия.

### 2. Handoffs between steps

Красный исход merge передаёт «the failing output in the note» (`skills/lado-checks/SKILL.md:76`). Шаг `implement` называет «a red check of the merge step» (`flows/feature.yaml:68`, `flows/fix.yaml:15`). Формат ноты указывает на роль: «note_body is your full report (your role, section 5)».

### 3. Done and outcomes

- **F3.1** [medium] `skills/lado-checks/SKILL.md:23`
  > | `src/lado/<mod>.py`, `src/lado/<pkg>/<mod>.py` | `uv run pytest tests/test_<mod>.py` and each test file that imports the module (`grep -rlw <mod> tests/`) |

  Строка не подходит к репозиторию LADO (`f1ab799`):
  1. Для модулей пакетов и для `hooks.py` нет файла `tests/test_<mod>.py`. Нет `test_hooks.py`, `test_claude.py`, `test_base.py`, `test_app.py`, `test_run.py`. Настоящие файлы — `test_providers.py` и `test_server_*.py`.
  2. `grep -rlw` ищет слово, а не импорт. Для `state` он находит 54 из 69 тестовых файлов, для `run` — 58, для `runtime` — 42.
  3. Найденные файлы смешивают unit, integration, ui и live. Integration- и ui-файлы в одном unit-прогоне молча отбрасываются, а итог всё равно печатает `N passed`. Пример: `pytest tests/test_state.py tests/integration/test_agents.py` → «91 passed, 29 deselected». Поэтому защита в `:32-34` («A `0 passed` … proves nothing») не срабатывает.

  Разработчик и ревьюер выберут разные наборы, а проверка «chose them right» становится спорной. Для ключевых модулей сужения почти нет — а именно оно цель R8.
  Fix: grep по импортам, только среди `.py`, например `grep -rlE 'lado(\.\w+)*\.<mod>\b|from lado(\.\w+)* import .*\b<mod>\b' tests --include='*.py'`. Добавить явное соответствие пакетов: `providers/` → `tests/test_providers.py`, `server/` → `tests/test_server*.py`. Попадания в `tests/integration/` и `tests/ui/` запускать с их `-m`, `tests/live/` оставить правилу live. Если файла нет или совпадений слишком много (больше ~10), запускать `make test`.
  Passes: 3/3

- **F3.2** [medium] `skills/lado-checks/SKILL.md:24`
  > | Code that drives processes, tmux, git, hooks, `lado mcp`, the server process or a provider (`runtime.py`, `hooks.py`, `mcp_server.py`, `tmux.py`, `state.py`, `loop.py`, `server/`, `providers/`) | also the matching `uv run pytest -m integration tests/integration/test_<x>.py` (fake agent, no LLM) |

  Ни один файл в `tests/integration/` не назван по модулю из списка. Там лежат `test_agents.py`, `test_flow_runs.py`, `test_session_loop.py`, `test_server_process.py` и другие. Значит, «the matching» — это догадка: одни разработчики не запустят ничего, и поломка проявится только в merge, через лишний круг с гейтом человека. Так же расплывчато «for each screen it touches» в `:25`.
  Fix: определить соответствие тем же grep по импортам среди `tests/integration/test_*.py`, запуск с `-m integration`. Или короткая карта «модуль → файлы integration».
  Passes: 3/3

- **F3.3** [medium] `skills/lado-checks/SKILL.md:50`
  > check is a finding. Do not run live tests: check that the report shows them green on

  У находки «пропущенная или неверная проверка» нет уровня, а вердикт ревьюера блокируют только Critical и Important (`agents/reviewer.md:82-83`). Ревьюер уже сам прогнал эти строки. Отправит ли он работу назад только ради повторного прогона — решает случай. Known hole 4 закрыта частично.
  Fix: «…is a finding: Important when your own run of that row is red or could not run, Minor when it is green».
  Passes: 3/3

- **F3.4** [medium] `skills/lado-checks/SKILL.md:18`
  > `make lint` (`make fmt` fixes most of it), then for each changed path every row that

  `make lint` требуется без условий. Но строка кита (`:29`, «| A kit, in its own repository (e.g. kit-lado-dev) | `lado kits check .` |») описывает репозиторий без Makefile, и там `make lint` красный. То же с «`make check`, always» в merge (`:75`). Ревьюер ответит «not Yes … a red check» или засчитает это как environment, и шаг встанет. Проходы дали low, low и medium; взят medium.
  Fix: «In the LADO repository, run `make lint`…». В строке кита написать «instead of `make lint`»; в merge, шаг 3, — «(in a kit's own repository, `lado kits check .`)». Другой вариант — удалить строку кита, если этот кит не работает с репозиториями китов.
  Passes: 2/3 (третий проход назвал то же для `:75`)

### 4. Independent verification

Цепочка: ревью → гейт `merge_ok` → полный `make check` в merge перед `--ff-only`. «Every branch, however small, is reviewed before it is merged» (`agents/supervisor.md:80`). Ревьюер сам прогоняет строки таблицы: «run the same rows of the table yourself» (`skills/lado-checks/SKILL.md:48`).

### 5. Contradictions

- **F5.1** [medium] `skills/lado-checks/SKILL.md:45` (state `implement`, визит после `merge` → `red`)
  > - **Developer**: after your last change, the checks the table names for your change, and
  > nothing more; live tests as above. Your report names each command and its last summary

  После `red` из merge упавший тест часто лежит вне изменённых путей разработчика — например, это взаимодействие с новым кодом из main. «nothing more» запрещает его перезапускать. А «When a check fails» требует «a fix with the / check green again» (`:99-100`), то есть полный `make check`. Одни разработчики прогонят полный `make check` вопреки R8. Другие не перезапустят упавший тест, и merge снова будет красным — ещё один круг через `merge_ok`.
  Fix: в пункте Developer: «…and nothing more, except after a `red` from merge: also the tests its note quotes, with their `-m`». Роли не меняются.
  Passes: 3/3

- **F5.2** [medium] `agents/developer.md:71`
  > claimed only with that fresh output (`verification-before-completion`).

  `verification-before-completion` говорит: «| "Partial check is enough" | Partial proves nothing |» и «- Relying on partial verification» (`SKILL.md:71`, `:56`). Целевые проверки частичны по замыслу, и нигде не сказано, какое правило главнее. Часть разработчиков прогонит полный набор, и R8 не выполнится. Проходы дали low и medium; взят medium.
  Fix: одна фраза в `lado-checks`, «Who runs what»: «for `verification-before-completion`, the rows of the table are the full check of your claim; the whole suite runs at merge».
  Passes: 2/3

### 6. Duplication

Команды проверок теперь есть только в `lado-checks`: `grep -nE "make |pytest" agents flows` ничего не находит. Правило платного прогона записано один раз (`skills/lado-checks/SKILL.md:39-40`), в супервизоре — только указатель (о его сужении см. F12.2).

### 7. When to call the human

Сбой окружения в merge: «tell the human what is missing and leave the step open» (`skills/lado-checks/SKILL.md:77-78`). В ревью — супервизору: «send the supervisor what is missing with `send_message`» (`agents/reviewer.md:86-87`). Seams: «without asking the human» (`agents/developer.md:35`).

### 8. Loops on a later visit

Петля `red: implement` ограничена гейтом `merge_ok` и `max_visits: 3` у `review`. При повторном визите `implement` называет, что исправлять: «a red check of the merge step». Повторный перезапуск упавших тестов описан в F5.1.

### 9. Concision and why

- **F9.1** [low] `flows/feature.yaml:19`
  > tested and at which seams (the seams under test), what is left out, and numbered acceptance criteria (AC-1, AC-2, ...), each

  Вставки не переформатированы: строки по 103–133 символа против ~90 в остальном тексте. То же в `flows/feature.yaml:74`, `flows/fix.yaml:21` и `agents/supervisor.md:24`. Это только вид, слов не прибавляет.
  Fix: переформатировать строки.
  Passes: 2/3

### 10. Skill descriptions

Описание `lado-checks` говорит, когда его брать: «Use before running checks or saying work on the LADO repo is done…». Каждый скилл объявлен своей папкой. `lado kits check` выдаёт «20 skills»: 19 внешних и 1 свой, все используются ролями. README и BLUEPRINT сверены.

### 11. Provider neutrality

В изменённом тексте нет инструментов CLI, моделей и конфигов. `PROVIDER=claude|kilo|opencode` — параметр Makefile самого LADO.

### 12. Safety and scope

- **F12.1** [high] `skills/lado-checks/SKILL.md:27`
  > | A test file | that file, with its `-m` |

  Роль разработчика велит расширять live-сценарий: «extend the live e2e scenario in `tests/live/`» (`agents/developer.md:42`). По этой строке разработчик запускает `uv run pytest -m live tests/live/test_live.py`. В LADO `tests/live/conftest.py:81` — это `@pytest.fixture(params=["claude", "kilo", "opencode"])`, то есть платная модель Claude запускается без «да» человека. Правило «да» в `:36-41` написано только для `make test-live PROVIDER=claude`. Строка новая: в 0.9.4 её не было.
  Fix: «A test file (not under `tests/live/`: that is a live test, below)». В абзаце Live tests: «any `-m live` run».
  Passes: 1/3, confirmed (`grep -n params tests/live/conftest.py` → `81:@pytest.fixture(params=["claude", "kilo", "opencode"])`)

- **F12.2** [medium] `skills/lado-checks/SKILL.md:55` (потерянное правило)
  > pushed commit. A release needs green CI on the release commit and `make test-live` on

  Удалённые строки на базе `58ea55f` держали правило «да» человека для любого платного прогона, включая релиз:
  - `agents/supervisor.md:80`: «- A paid `make test-live PROVIDER=claude` needs the human's yes every time, also inside a»
  - `skills/lado-checks/SKILL.md:20`: «… Release: on main. For Claude, ask the human, through the supervisor, every time: it uses a paid model. |»

  Сейчас `agents/supervisor.md:79` («A developer's request for a paid live test goes to the human every time») и `skills/lado-checks/SKILL.md:39` («the developer asks the supervisor») покрывают только разработчика. `make test-live` на main при релизе без `PROVIDER` запускает все провайдеры, включая Claude, и «да» никто не спрашивает. Это расходится с BLUEPRINT R9: «A paid live run with a real model needs the human's yes every time».
  Fix: в `lado-checks`, Live tests: «whoever runs it — the developer through the supervisor, the supervisor at a release — gets the human's yes first».
  Passes: cut-rule check (найдено и всеми тремя проходами)

## Known holes

| Known hole | Finding, or how the kit handles it |
|---|---|
| 1. Red check sent back with no environment cause considered | Закрыто. Merge: «Report `red` … only for code or test … For environment, tell the human what is missing and leave the step open» (`skills/lado-checks/SKILL.md:76-78`). Ревью: «an environment failure is no finding» (`agents/reviewer.md:86`). |
| 2. Work outside a flow, merge without a gate | Закрыто: «Any code goes through `fix` or `feature`, so every merge into main has a review and the human's `merge_ok`» (`agents/supervisor.md:61-63`). Платный прогон без «да» — F12.1, F12.2. |
| 3. Path outside the run's worktree | Новых случаев нет. Лог «outside the tree» — так задумано. `--ff-only` «In your repo, on main» — только в merge. |
| 4. Verdict without a severity threshold | Вердикт закрыт: «The answer is Yes when no Critical or Important finding is open» (`agents/reviewer.md:82-83`). Нет уровня для находки «пропущенная проверка»: F3.3. |
| 5. Dependency skill that writes or asks where its role must not | Закрыто: `brainstorming` и `domain-modeling` убраны; `tdd` перекрыт фразой «without asking the human» (`agents/developer.md:35`). Новое противоречие со скиллом без записи и вопросов: `verification-before-completion` против целевых проверок, F5.2. |

## Not traced

Нет. Каждая роль, шаг, гейт и скилл есть в таблице трассировки BLUEPRINT.md §3. Единственная жёлтая мера (похожие абзацы) обоснована в §4.

## Previous findings

| 0.9.4 | Статус | Доказательство |
|---|---|---|
| F1.1 `domain-modeling` у архитектора | RESOLVED | `agents/architect.md:36`: «Use `codebase-design` for»; в `skills:` его нет |
| F3.1 ревьюер без классификации красного | RESOLVED | `agents/reviewer.md:85-86`: «Classify a red check as `lado-checks`, "When a check fails", says: an environment failure is no finding» |
| F3.2 нет порога вердикта | RESOLVED | `agents/reviewer.md:82`: «The answer is Yes when no Critical or Important finding is open» |
| F3.3 таблица устарела | RESOLVED | `skills/lado-checks/SKILL.md:58-59`: «`make check` runs `make lint`, `make test-js`, `make web` and `make browser`, then one parallel pytest run…»; формы `-m integration` / `-m ui` в `:24-25`. Отдельной строки `make test-ui` нет — по причине автора (R8). Новые неточности таблицы — F3.1, F3.2. |
| F5.1 «`make test` After any change» | RESOLVED | `:30`: «Code no row above matches \| `make test` (all unit tests)» |
| F5.2 `brainstorming` у супервизора | RESOLVED | нет в `skills:` и в §2 |
| F5.3 seams в `tdd` | RESOLVED | `agents/developer.md:33-35`: «The seams under test the brief or design names count as confirmed; if none are named, pick them yourself…» |
| F6.1 платный test-live в 4 местах | RESOLVED, но при сокращении правило сузилось | одно место (`lado-checks:39-40`) плюс указатель; потеря — F12.2 |
| F6.2 done-условие flows привязано к §5 роли | RESOLVED | `flows/feature.yaml:72`: «checks `lado-checks` names for your change are green» |
| F6.3 developer.md:68 пересказ | RESOLVED | `agents/developer.md:70`: «run the checks `lado-checks` names for your change» |
| F10.1 родительские папки паков | RESOLVED | `kit.yaml`: каждая папка — отдельный скилл; `lado kits check`: «20 skills» |

Список автора «fixed» сверен с файлами: все пункты на месте.

## Cut rules

| Removed rule (file:line at base) | Where it is now |
|---|---|
| Платный test-live: «да» человека every time (`agents/supervisor.md:80`; `skills/lado-checks/SKILL.md:20`, «Release: on main. For Claude, ask the human…») | Сужено до разработчика (`lado-checks:39`, `supervisor.md:79`); для релиза потеряно — **F12.2** |
| Developer: `make check` / `make lint` / test-live (`agents/developer.md:68-71`) | `skills/lado-checks/SKILL.md:45-47` и таблица `:21-30` |
| Report names each command and its last summary line (`lado-checks:22-23`) | `lado-checks:46-47` |
| `make test-integration` после runtime/hooks/mcp_server/tmux/state/провайдера (`lado-checks:17`) | `lado-checks:24` (неточность — F3.2) |
| `make test-js` после plugin (`lado-checks:18`) | `lado-checks:26` (файл теперь `opencode_plugin.js`, проверено в LADO) |
| Не-код → `make lint` only; кит → `lado kits check` (`lado-checks:31-32`) | `lado-checks:28-29` |
| `.md` под `src/` — это код (`lado-checks:40-41`) | `lado-checks:28`: «Any other path is code, also a `.md` under `src/` or `tests/`» |
| test-live: when, once per round, developer, DONE_WITH_CONCERNS (`lado-checks:33-36`) | `lado-checks:36-41` |
| Ревьюер не запускает live, проверяет отчёт (`lado-checks:36-38`) | `lado-checks:50-51` |
| While working — only tests next to your change; final check once after last change (`lado-checks:29-30`) | `lado-checks:11-13`, `:45` |
| Release: test-live on main + green CI (`lado-checks:51-52`) | `lado-checks:54-56` |
| Ревьюер: `make check` с SHA, повторное использование результата; правило пропуска в merge; `git rev-parse HEAD` (`reviewer.md:31-35`, `lado-checks:42-50`, `:65`) | Удалены по решению человека (R8): полный `make check` всегда в merge |
| `brainstorming`, `domain-modeling` | Удалены по плану (F5.2, F1.1 отчёта 0.9.4) |

## Missed earlier

Нет. Все новые находки — в тексте, который изменил diff.

## Left by the plan

- Похожие абзацы (бюджет жёлтый: review 92%, implement 79%, вступления architect/reviewer 77%). Причина человека из плана: две flow задуманы так (R2, R6), шаги различаются источником AC, каждый `do` должен читаться сам по себе; записано в BLUEPRINT.md §4.
