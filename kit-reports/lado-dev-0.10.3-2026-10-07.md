# Kit report: lado-dev 0.10.3

- Дата: 2026-10-07
- Кит: worktree run'а `improve/lado-dev-4` (`.lado/worktrees/kit-lado-dev/improve-lado-dev-4`), ветка `lado/kit-lado-dev/improve-lado-dev-4`
- Коммит: `683a227`
- Оценивал: критик kit-builder (слои a и b)
- Режим: повторная оценка, визит 1 шага `evaluate` в `improve`, нужен вердикт.
  - Предыдущий отчёт: `kit-reports/lado-dev-0.10.2-2026-10-07.md`. План называет для него коммит `3743a65`, он же база.
  - Изменение: `git diff 3743a65 -- kit.yaml README.md BLUEPRINT.md agents flows skills`. Затронуты 4 файла: kit.yaml, BLUEPRINT.md, `agents/supervisor.md`, `skills/lado-checks/SKILL.md`.
- Проходы. Три независимых суб-агента прошли по изменённому тексту и тому, что его окружает. Каждый получил рубрику, кит, diff, цель плана и репозиторий LADO (`3392bd8`) для проверки фактов.
  - Проход 1 начал с kit.yaml и ролей.
  - Проход 2 начал с flows и прослеживал сценарии: merge (flaky, red, environment, коммит одного BACKLOG.md), сбой среды у разработчика, релиз до тега.
  - Проход 3 начал с `lado-checks` и применил строки таблицы к реальным путям LADO.
  - Вырезанные правила проверил сам критик по diff.
  - Полный проход (Re-evaluation 5) по четырём затронутым файлам целиком сделал четвёртый суб-агент. Его находки критик сверил с файлами и с LADO.
  - Отброшена 1 однопроходная находка: «после `fix` неясно, что "that commit" (`:97`) — новая голова main». Вред не показан: релиз «goes on from pushing main», а порядок в `:95` снова начинается с «push main; wait for green CI on that commit».
  - Как подтверждённые оставлены 2 однопроходные находки (F5.15, F3.22).

Находки — кандидаты, которые взвешивает человек. Это не оценка «прошёл / не прошёл».

## Card

| Слой | Результат |
|---|---|
| a. `lado kits check` | OK, 0 предупреждений |
| a. Бюджет | жёлтый. Все числовые меры зелёные. Жёлтые только 3 пары похожих абзацев (92%, 79%, 77%), они обоснованы в BLUEPRINT.md §4. Слов: developer 800, supervisor 800 (порог для лида 1000), reviewer 785, architect 574. |
| a. Flows | нарисованы 2; совпадают со скелетами BLUEPRINT.md (`same as the plan`) |
| b. Рубрика | 8 находок в изменённом тексте: 0 high, 4 medium, 4 low. Ещё 1 в «Missed earlier» (low). Без находок в изменённом тексте 9 из 12 критериев (1, 4, 6, 7, 8, 9, 10, 11, 12). Из 13 находок отчёта 0.10.2 все 13 RESOLVED; у трёх есть остаток, он вынесен в новые находки. |
| Охват | повторная оценка изменённого текста плюс полный проход по 4 файлам, которые затронул diff |
| Stop rule | не выполнено: 4 medium в изменённом тексте. Это совет для гейта релиза, не блок. |

Счёт относится только к тому, что указано в строке «Охват». Сравнивать его со счётом полной оценки нельзя.

### `lado kits check .`

```
lado-dev: OK (3 agents, 20 skills, 4 packs and 2 flows)
```

### Budget script (exit status 0)

```
# Complexity budget: lado-dev 0.10.3

| Measure | Where | Value | Green / yellow up to | Zone |
|---|---|---|---|---|
| Worker roles (not supervisor) | kit | 3 | 3 / 5 | green |
| Work steps in a flow | flows/feature.yaml | 5 | 5 / 8 | green |
| Work steps in a flow | flows/fix.yaml | 3 | 5 / 8 | green |
| Gates in a flow | flows/feature.yaml | 2 | 2 / 3 | green |
| Gates in a flow | flows/fix.yaml | 1 | 2 / 3 | green |
| Words in a role prompt | agents/architect.md | 574 | 800 / 1500 | green |
| Words in a role prompt | agents/developer.md | 800 | 800 / 1500 | green |
| Words in a role prompt | agents/reviewer.md | 785 | 800 / 1500 | green |
| Words in the lead's prompt | agents/supervisor.md | 800 | 1000 / 1500 | green |
| Own skills | kit | 1 | 5 / 10 | green |
| MCP servers | kit | 0 | 2 / 4 | green |

## Similar paragraphs (one rule, one place; 55% similar or more)

- yellow: 92% similar: flows/feature.yaml: state "review": "Review the run's branch against main, read-only, as your role describes: the ..." ~ flows/fix.yaml: state "review": "Review the run's branch against main, read-only, as your role describes: the ..."
- yellow: 79% similar: flows/feature.yaml: state "implement": "Implement the design (the note from design, as the human approved it) in the ..." ~ flows/fix.yaml: state "implement": "Implement the task in the run's worktree, test-first, as your role describes...."
- yellow: 77% similar: agents/architect.md: "Most reviews come as a step of a flow run (a message from `lado`): the design..." ~ agents/reviewer.md: "Most reviews come as a step of a flow run (a message from `lado`): the step s..."

Overall: yellow
```

### Flows

![feature](lado-dev-0.10.3-2026-10-07/feature.svg)
![fix](lado-dev-0.10.3-2026-10-07/fix.svg)

```
same as the plan in BLUEPRINT.md
(exit status 0)
```

## Fix first

Вердикт `approved`. `lado kits check` прошёл без ошибок, красных мер нет, жёлтые обоснованы, `--compare` различий не нашёл, high-находок нет. Все 13 находок 0.10.2 закрыты. Ниже то, что осталось в новом тексте релиза и выбора тестов; ни один пункт не блокирует.

1. Релиз: красный CI или live (или пропущенный провайдер) сначала классифицировать; в `fix` идут только code и test (F3.19).
2. Релиз: «Live tests» `:67` («…or none») спорит с релизом `:98` («every other provider») (F5.14).
3. Строка `web/`: имена UI-тестов не совпадают с именами экранов (`Agents.tsx` → `test_agents_tab.py`) (F3.20).
4. Перечень путей для live не содержит `tmux.py`, `loop.py` и хелперов изоляции live-тестов — вопрос человеку 1 (F3.21).

## Findings

### 1. Role boundaries

Изменение ролей не меняет прав: §3 супервизора разрешает вне flow только «the writes on main `lado-checks` allows (version, BACKLOG.md)» (`agents/supervisor.md:62`), а `lado-checks` отдаёт их супервизору («**Release** (the supervisor, only when the human asks)», `:93`; «Outside a run, the supervisor adds them on main», `:187`).

### 2. Handoffs between steps

Flows не менялись. Нот без источника не появилось: `red` из merge по-прежнему несёт «the failing output in the note» (`:126`).

- **F2.2** [low] `BLUEPRINT.md:179`
  > | 2026-10-07 | 0.10.3 | Release of LADO: live tests of every provider (on a no to Claude, every other one); red CI or live after pushing main → no tag, fix through `fix`, push main again with the same version. Developer: a non-live check that cannot run for an environment cause → BLOCKED. Targeted tests: a changed conftest or helper under `tests/` runs its folder's `test_*.py`; a `web/` file that names no screen runs all of `tests/ui/`; every row that matches a path; more than 10 unit hits counted over both searches. Live tests needed for `providers/`, `hooks.py`, `mcp_server.py`, `runtime.py`, `agent_env.py`, `tests/live/`. Merge Done and a BACKLOG.md-only commit agree. R9 updated. | F3.12–F3.18, F5.11–F5.13, F6.3, F9.1, F10.1 of `kit-reports/lado-dev-0.10.2-2026-10-07.md`; the human (triage of `improve/lado-dev-4`, questions 1–2). |

  Журнал называет «Developer:», но правило лежит в `lado-checks:150-152`, а `agents/developer.md` не менялся. Читатель журнала ищет его не там. На агентов не влияет.
  Fix: «`lado-checks` (When a check fails): the developer reports BLOCKED …».
  Passes: full pass, confirmed (`git diff --stat 3743a65 HEAD` не содержит `agents/developer.md`)

### 3. Done and outcomes

- **F3.19** [medium] `skills/lado-checks/SKILL.md:100`
  > release. Red CI or live tests: no tag; tell the human, quoting the failing line; the fix

  Known hole 1 на релизе. Любой красный результат после push main идёт в `fix`, без классификации: сбой раннера CI, flaky, провайдер с незалогиненным CLI или rate limit бесплатной модели («"opencode/big-pickle" is often rate-limited», LADO `tests/live/conftest.py:11`). «When a check fails» называет, кто действует, для developer, reviewer и merge (`:138-139`), но не для релиза. Пропущенный провайдер — «a skipped provider is not green» (`:70`): тега нет, но и «red» это не названо, так что следующий шаг супервизора неясен (ждать, тегнуть, открыть `fix`). `fix` по среде разработчик закончит только BLOCKED.
  Fix: «Red CI or live tests, or a skipped provider: classify as "When a check fails" (flaky: one rerun, a paid one with a yes). Code or test: no tag, tell the human with the failing line, the fix goes through `fix`, then … Environment: tell the human what is missing and rerun.»
  Passes: 3/3 и full pass (проход 2 разделил случай «пропущен» на отдельную находку на `:70`; одно место, одна причина — объединено)

- **F3.20** [medium] `skills/lado-checks/SKILL.md:29`
  > | `web/` | `make web` (also fails on a stale `web/openapi.json`), `make browser`, then the UI tests of each screen it changes, by file name (`tests/ui/test_<screen>.py`); for a file that names no screen (e.g. `Shell.tsx`, `App.tsx`, `styles.css`, `tokens.css`, `api.ts`, `live.ts`, `Settings.tsx`, `Team.tsx`, `Sessions.tsx`), all of `tests/ui/` with `-m ui` |

  F3.13 закрыт для файлов без экрана, но правило «by file name» не совпадает с LADO: `Agents.tsx` → `test_agents_tab.py`, `Flows.tsx` → `test_flows_tab.py`, `Kits.tsx` → `test_kits_page.py`, `Terminals.tsx` → `test_terminal_panel.py`, `GateCard.tsx` → `test_gates.py`. `test_agents.py` нет. Один агент запустит `test_agents_tab.py`, другой решит «names no screen» и запустит все `tests/ui/`; ревьюер запишет Minor «wrong check». Против цели «a short rule both developer and reviewer apply alike» (`:49-50`).
  Fix: «the `tests/ui/test_*.py` whose name contains the screen's name (e.g. `Agents.tsx` → `test_agents_tab.py`); none: all of `tests/ui/` with `-m ui`».
  Passes: 2/3 и full pass (импакт по проходам low, medium, low — взят medium; `ls tests/ui` в LADO `3392bd8`)

- **F3.21** [medium] `skills/lado-checks/SKILL.md:60` (cut-rule check)
  > CLIs and models. They run only when the change touches `src/lado/providers/`,

  Перечень заменил «how agents get their input» (база `:58`). В LADO текст в CLI агента вводит `tmux.send_text` (`src/lado/tmux.py:176`) с правилом только для Claude: «Claude Code reads a backslash before» Enter as a line break (`:180`). Его вызывают `runtime.py:1068,1094`; `tests/live/test_live.py:16` импортирует `loop, providers, runs, runtime, state, terminal, tmux`. Правка `tmux.py` или `loop.py` теперь не запускает live, а `make check` на merge их не гоняет: первым настоящий CLI увидит её на релизе. Так же `tests/agent_helpers.py` (`refuse_unless_isolated`, импорт в `tests/live/conftest.py:32`) и `tests/conftest.py`: по новой строке `:28` для них `make test`, live не нужен, хотя это охрана изоляции платного прогона. Перечень — решение человека в triage, поэтому вопрос 1.
  Fix: добавить в перечень `src/lado/tmux.py`, `src/lado/loop.py`, `tests/conftest.py`, `tests/agent_helpers.py` — или оставить как есть по ответу человека.
  Passes: cut-rule check и full pass, проход 3 (3 источника)

- **F3.22** [low] `skills/lado-checks/SKILL.md:28`
  > | A `conftest.py` or helper under `tests/`, outside `tests/live/` | each `test_*.py` in its folder, with its `-m`; for one directly in `tests/`, `make test` |

  Для `tests/ui/` условие «(after `make web` and `make browser`)» стоит только в абзаце про попадания поиска (`:46`), а не в строках `:27` и `:28`. В свежем worktree `src/lado/server/static/` нет (LADO `.gitignore:13`), и UI-тесты получают 503 «The web UI's bundle is missing» (`app.py:641`). Разработчик доложит BLOCKED по среде, ревьюер после `make web` увидит зелёное.
  Fix: в `:45-46` — «every `-m ui` run comes after `make web` and `make browser`».
  Passes: 1/3, confirmed (`grep -n static .gitignore` → `13:src/lado/server/static/`; `app.py:641` → `PlainTextResponse(f"The web UI's bundle is missing: …", 503)`)

### 4. Independent verification

Flows не менялись: `review` (`max_visits: 3`), гейт `merge_ok`, полный `make check` в `merge`. Тег — только после зелёного CI и live: «The tag needs green CI and green live tests on that commit» (`:97`).

### 5. Contradictions

- **F5.14** [medium] `skills/lado-checks/SKILL.md:67`
  > On a no, run only the other providers the change needs, one `make test-live

  Абзац «Live tests» прямо обращён к релизу («and at a release», `:63`; «the supervisor at a release», `:66`) и разрешает «…or none». Новый релиз требует «run every other provider» (`:98`). До правки `:92` ссылался сюда «as above», теперь две разные нормы для одного случая. Супервизор, прочитавший `:67` первым, может перед необратимым тегом прогнать часть провайдеров или ни одного. Это остаток F5.12.
  Fix: «During work, on a no, run only …» или в `:63` «and at a release (every provider: Release)».
  Passes: 3/3 и full pass (импакт по проходам medium, medium, low — взят medium)

- **F5.15** [low] `skills/lado-checks/SKILL.md:48`
  > unit hits. Integration and UI tests run only as hits, and with none, none run: the full

  Две новые строки запускают integration и UI не как попадания: `:28` (хелпер в `tests/integration/` → все его `test_*.py`) и `:29` (файл без экрана → все `tests/ui/`). Буквально `:48` их отменяет. Решение человека «только по попаданиям» относится к модулям, но фраза этого не говорит.
  Fix: «For a changed module, integration and UI tests run only as hits; …».
  Passes: 1/3, confirmed (`:28` «each `test_*.py` in its folder, with its `-m`» и `:29` «all of `tests/ui/` with `-m ui`» против `:48`)

- **F5.16** [low] `skills/lado-checks/SKILL.md:101`
  > goes through `fix`, then the release goes on from pushing main with the same version.

  Супервизор держит `fix` для «a small, clearly scoped change whose acceptance criteria you can state up front» (`agents/supervisor.md:20`). Красный релиз может требовать дизайна, а `:101` всё равно шлёт в `fix`, мимо architect и `design_ok`.
  Fix: «goes through `fix` or `feature` (supervisor §1)».
  Passes: full pass, confirmed

### 6. Duplication

F6.3 закрыт: R9 (`BLUEPRINT.md:36-43`) и «Release» (`:93-102`) теперь говорят одно и то же. Новых повторов нет; спор `:67` и `:98` — F5.14.

### 7. When to call the human

Нарушений в изменённом тексте нет. Красный релиз: «tell the human, quoting the failing line» (`:100`). Сбой среды у разработчика — BLOCKED, его супервизор передаёт человеку (`supervisor.md:77-79`). Каждый push — «Each push needs the human's yes» (`:102`).

### 8. Loops on a later visit

Flows не менялись. Перезапуск на merge ограничен: «rerun the whole check once» (`:121-122`); повторный merge после неудачного ff снова проходит шаг 3 (`:129-130`).

### 9. Concision and why

Нарушений в изменённом тексте нет. F9.1 закрыт (`:174-177`). Роли — ровно по 800 слов. Правка §3 супервизора убрала только примеры («a question, an investigation, a look at a branch»), не правило.

### 10. Skill descriptions

F10.1 закрыт: «when you release LADO» в описании (`:3`). Списки скиллов не менялись.

### 11. Provider neutrality

Имён инструментов CLI нет. `PROVIDER` — переменная Makefile LADO.

### 12. Safety and scope

Нарушений в изменённом тексте нет: тег — после зелёного CI и live; «Red CI or live tests: no tag» (`:100`); каждый push — с «да»; вне flow на main пишется только версия и BACKLOG.md (`supervisor.md:62`).

## Known holes

| Known hole | Finding, or how the kit handles it |
|---|---|
| 1. Red check sent back with no environment cause considered | Разработчик закрыт: «The developer reports BLOCKED; only a live provider skipped for it is DONE_WITH_CONCERNS» (`lado-checks:150-152`). Merge: «classify the second run's first error, as code, test or environment» (`:125`). Ревьюер: `:86`. Открыто на релизе — **F3.19**. |
| 2. Work outside a flow, merge without a gate | «Outside a flow goes only read-only work, … and the writes on main `lado-checks` allows (version, BACKLOG.md)» (`supervisor.md:61-62`); «every merge into main has a review and the human's `merge_ok`» (`:63-64`); «Each push needs the human's yes» (`lado-checks:102`). |
| 3. Path outside the run's worktree | Закрыто: изменение добавило только пути от корня репозитория; лог «outside the tree» (`:108`); релиз идёт на main в репозитории супервизора, как и задумано. |
| 4. Verdict without a severity threshold | Закрыто: «The answer is Yes when no Critical or Important finding is open» (`agents/reviewer.md:82-83`); «Important when your own run of it is red for code or test, otherwise Minor» (`lado-checks:85-86`). |
| 5. Dependency skill that writes or asks where its role must not | Закрыто: изменение не добавило скиллов и не меняло `skills:` ролей; read-only роли только перечисляют находки в **Found on the way** (`lado-checks:179-181`). |

## Not traced

Нет. Каждая роль, шаг, гейт и скилл есть в трассировке BLUEPRINT.md §3. Жёлтые пары обоснованы в §4. `--compare`: «same as the plan in BLUEPRINT.md».

## Previous findings

Находки отчёта `kit-reports/lado-dev-0.10.2-2026-10-07.md`:

| Находка | Статус | Доказательство |
|---|---|---|
| F3.12 правка conftest/хелпера | RESOLVED | `:28` «A `conftest.py` or helper under `tests/`, outside `tests/live/` \| each `test_*.py` in its folder, with its `-m`; for one directly in `tests/`, `make test`». Условие `make web` для `tests/ui/` — F3.22. |
| F3.13 `web/`-файл без экрана | RESOLVED | `:29` «for a file that names no screen (…), all of `tests/ui/` with `-m ui`». Несовпадение имён экранов — F3.20. |
| F3.14 «its row» | RESOLVED | `:21-22` «every row that matches it»; `:27` «A `test_*.py` file outside `tests/live/`». |
| F3.15 больше 10 попаданий | RESOLVED | `:47` «more than 10 unit hits (both searches together)». |
| F3.16 красный CI/live после push main | RESOLVED | `:100-101` «Red CI or live tests: no tag; tell the human, quoting the failing line; the fix goes through `fix`, then the release goes on from pushing main with the same version». Классификация причины — F3.19. |
| F3.17 Done merge и «first error» | RESOLVED | `:125` «classify the second run's first error»; `:132-134` «…or on its parent when the last commit only adds BACKLOG.md entries». |
| F3.18 «how agents get their input» | RESOLVED | `:60-62` перечень путей. Неполнота перечня — F3.21. |
| F5.11 сбой среды у разработчика | RESOLVED | `:150-152` «The developer reports BLOCKED; only a live provider skipped for it is DONE_WITH_CONCERNS ("Live tests")»; `supervisor.md:77` BLOCKED оставляет шаг открытым. |
| F5.12 live на релизе | RESOLVED | `:96` «the live tests of every provider (`make test-live`)»; `:98` «run every other provider». Старая норма в `:67` — F5.14. |
| F6.3 R9 | RESOLVED | `BLUEPRINT.md:36-40` «the live tests of every provider must be green … declines it at a release, every other provider's live tests must be green». |
| F9.1 длинная строка | RESOLVED | `:174-177` переформатировано. |
| F5.13 §3 супервизора | RESOLVED | `supervisor.md:61-62` «and the writes on main `lado-checks` allows (version, BACKLOG.md)». |
| F10.1 описание | RESOLVED | `:3` «when you release LADO». |
| Похожие абзацы (жёлтый бюджет) | оставлено планом | BLUEPRINT.md §4 |

Список fixed / not fixed автора сверен с файлами и совпадает. Замечание автора про `supervisor.md:79` (BLOCKED там уже оставляет шаг открытым) верно.

## Cut rules

| Removed rule (file:line at base) | Where it is now |
|---|---|
| «for each changed path its row» (`lado-checks:21`) | `:21-22` «every row that matches it» |
| «A test file outside `tests/live/`» (`:26`) | `:27` для `test_*.py`; прочие файлы под `tests/` — `:28` и `:30` |
| «the UI tests of each screen it changes, by file name» (`:27`) | `:29` без изменений плюс запасной вариант |
| «more than 10 unit hits» (`:45`) | `:47` с уточнением |
| «a provider, hooks, the MCP server, how agents get their input or `tests/live/`» (`:57-58`) | `:60-62` перечень путей по решению человека; `tmux.py`, `loop.py` в него не вошли — **F3.21** |
| «the live tests, as above» и «the other providers' live tests must be green» (`:92-94`) | `:96-99` |
| «classify the first error as "When a check fails" says» (`:118`) | `:125` |
| Done merge (`:125`) | `:132-134` с исключением |
| «Report what is missing and the command that showed it» (`:141`) | `:149-150` без изменений |
| `supervisor.md:61-62` «(a question, an investigation, a look at a branch): start a worker with `spawn_worker(role=...)` and a self-contained brief» | `:61-62` «by a worker from `spawn_worker(role=...)` with a self-contained brief»; убраны только примеры |
| R9: «the live tests of the other providers the change needs must be green» (`BLUEPRINT.md:39`) | `BLUEPRINT.md:41-42` по плану |

Потерянных правил нет. Одно заменено неполным перечнем — F3.21 (вопрос 1).

## Missed earlier

Находки полного прохода в тексте, который diff не трогал. Вердикт они не блокируют и в stop rule не учитываются.

- **F3.23** [low] `skills/lado-checks/SKILL.md:63`
  > checks above, and at a release. A change to one provider runs only that provider's

  Не сказано, что делать с общими файлами провайдеров: `opencode_family.py` и `opencode_plugin.js` обслуживают kilo и opencode, `base.py` и `__init__.py` — всех, включая платный Claude.
  Fix: «A change to one provider's own file runs only that provider's; `opencode_family.py` or `opencode_plugin.js` runs kilo and opencode; any other file under `providers/` runs every provider.»
  Passes: full pass, confirmed (`ls src/lado/providers/` в LADO `3392bd8`)

## Left by the plan

- Похожие абзацы (бюджет жёлтый: review 92%, implement 79%, вступления architect/reviewer 77%). Причина человека: две flow задуманы так (R2, R6), шаги различаются источником AC, каждый `do` читается сам. Записано в BLUEPRINT.md §4.
- «Integration и UI только по попаданиям» — принято человеком в 0.10.2, план не меняет. F5.15 касается только формулировки, не решения.

## Questions for the human

1. Дополнить перечень путей для live (F3.21)? Сейчас: `providers/`, `hooks.py`, `mcp_server.py`, `runtime.py`, `agent_env.py`, `tests/live/`. Вне перечня: `tmux.py` (`send_text` с правилом для Claude про обратный слэш перед Enter), `loop.py` (рассылка сообщений агентам), `tests/agent_helpers.py` и `tests/conftest.py` (изоляция live-прогона от настоящего LADO). Правка в них всплывёт только на релизе, в платном прогоне.
   Рекомендация: добавить `src/lado/tmux.py` и `src/lado/loop.py` — это то, что раньше значило «how agents get their input». Хелперы изоляции — по вашему решению; без них сломанная изоляция впервые проявится в платном прогоне.
