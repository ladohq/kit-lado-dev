# Kit report: lado-dev 0.10.1

- Дата: 2026-10-06
- Кит: worktree run'а `improve/lado-dev-2` (`.lado/worktrees/kit-lado-dev/improve-lado-dev-2`), ветка `lado/kit-lado-dev/improve-lado-dev-2`
- Коммит: `a6d6cd7`
- Оценивал: критик kit-builder (слои a и b)
- Режим: повторная оценка, визит 1 шага `evaluate` в `improve`, нужен вердикт.
  - Предыдущий отчёт — `kit-reports/lado-dev-0.10.0-2026-10-06.md`. План называет для него коммит `a466a55`, это и есть база.
  - Изменение: `git diff a466a55 -- kit.yaml README.md BLUEPRINT.md agents flows skills`. Затронуты 6 файлов: kit.yaml, BLUEPRINT.md, 3 роли, `lado-checks`.
- Проходы: три независимых суб-агента по изменённому тексту и тому, что его окружает. Каждому дали рубрику, кит, diff, зависимые скиллы и репозиторий LADO (`fc131c7`) для проверки фактов.
  - Проход 1 начинал с kit.yaml и ролей.
  - Проход 2 начинал с flows и прослеживал сценарии: правка `providers/kilo.py`, «нет» на Claude, flaky на merge, UI-дизайн с правкой `ui.md`, отмена run'а, релиз.
  - Проход 3 начинал с `lado-checks` и применил процедуру к `providers/claude.py`, `server/feed.py`, `tmux.py`, `loop.py`, `web/` (grep по LADO, без тестов).
  - Вырезанные правила проверял сам критик. Потерянных нет.
  - Полный проход (Re-evaluation 5) по шести затронутым файлам целиком, с зависимыми скиллами, делал четвёртый суб-агент. Его находки критик подтвердил в файлах и в LADO.
  - Отброшены 2 однопроходные находки (low): правка `ui.md` не оформлена как AC; в списке отчёта разработчика (§5) нет пункта **Found on the way** для работы вне run'а (§4 и так говорит это явно).
  - Однопроходных находок, оставленных как подтверждённые, нет.

Находки — кандидаты, которые взвешивает человек. Это не оценка «прошёл / не прошёл».

## Card

| Слой | Результат |
|---|---|
| a. `lado kits check` | OK, 0 предупреждений |
| a. Бюджет | жёлтый. Все числовые меры зелёные. Жёлтые только 3 пары похожих абзацев (92%, 79%, 77%), обоснованы в BLUEPRINT.md §4. Слов в ролях: developer 800, supervisor 800 (порог 1000 для лида), reviewer 785, architect 574. |
| b. Рубрика | 10 находок в изменённом тексте: 0 high, 6 medium, 4 low. Ещё 2 в «Missed earlier» (2 medium). Без находок в изменённом тексте 9 из 12 критериев (1, 2, 4, 7, 8, 9, 10, 11, 12). Из 15 находок отчёта 0.10.0: 14 RESOLVED, F3.3 STILL OPEN (его интеграционная половина). |
| Охват | повторная оценка изменённого текста плюс полный проход по 6 файлам, которые затронул diff |
| Stop rule | не выполнено: 6 medium в изменённом тексте. Это совет для гейта релиза, не блок. |

Счёт относится только к тому, что указано в строке «Охват». Сравнивать его со счётом полной оценки нельзя.

### `lado kits check .`

```
lado-dev: OK (3 agents, 20 skills, 4 packs and 2 flows)
```

### Budget script (exit status 0)

```
# Complexity budget: lado-dev 0.10.1

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

## Fix first

Вердикт `approved`: `lado kits check` без ошибок, красных мер нет, жёлтая обоснована, high нет. Ниже — что стоит исправить до релиза или следующим раундом. Пункты 1–4 правятся в `skills/lado-checks/SKILL.md`, абзацы о поиске тестов (`:27`, `:44-49`).

1. Ограничить новое предложение `:47-49` только попаданиями в conftest и хелперы. Тестовые файлы, которые нашёл поиск, по-прежнему идут со своим `-m` (F5.8).
2. Однозначно сказать, что идёт из integration при правке провайдера: хелпер `fake_provider.py`, «all of them when it finds none» в `:27` (F3.3, STILL OPEN). В `:27` заменить перечень модулей на «any module the search ties to an integration test» (F3.7).
3. Провайдер, пропущенный из-за окружения: у разработчика DONE_WITH_CONCERNS, у ревьюера это concern для `merge_ok`, а не Important (F3.8).
4. При «нет» на Claude — live-тесты только тех других провайдеров, которые нужны изменению, по одному `make test-live PROVIDER=<name>` (F3.9).
5. Шаг merge: сначала один перезапуск всего `make check`, потом классификация. Коммит одного BACKLOG.md после зелёного прогона не требует новой проверки (F3.10, F3.11).

## Findings

### 1. Role boundaries

Новое правило: «Only the developer fixes; a reviewer reports the class, the merge step acts as its step 3 says» (`skills/lado-checks/SKILL.md:126`). Разработчик вне run'а передаёт свои пункты супервизору: «the supervisor records them» (`agents/developer.md:66`). Пересечений прав нет.

### 2. Handoffs between steps

В изменённом тексте нот без источника нет. Дизайн несёт «what to change in `docs/design/ui.md`» (`agents/supervisor.md:54`). `red` из merge несёт «the failing output in the note».

### 3. Done and outcomes

- **F3.3** [medium] STILL OPEN `skills/lado-checks/SKILL.md:46`
  > `tests/ui/conftest.py` or `tests/integration/conftest.py` does not pull in its whole layer:

  Половина про UI исправлена, половина про integration — нет. Исключение называет только `conftest.py`, а `:44` по-прежнему говорит «A hit that is not a `test_*.py` file (a `conftest.py`, a helper) stands for every test file». Для любого модуля `providers/` поиск с `P=lado`, `<mod>=providers` находит в integration только `tests/integration/conftest.py` и хелпер `tests/integration/fake_provider.py` (`from lado import cli, providers, state`), а `test_*.py` не находит ни одного.

  Отсюда три прочтения:
  - по `:44` хелпер даёт весь слой;
  - по `:27` («or all of them … when it finds none») — тоже весь слой;
  - по `:48` («then only the test files the search finds») — ничего.

  Changelog BLUEPRINT.md при этом заявляет «no UI/integration suites pulled by a provider change». Ещё неясно, включает ли «its folder» подпапки для хелпера `tests/agent_helpers.py`.
  Fix: «A conftest or helper under `tests/integration/` or `tests/ui/` (such as `fake_provider.py`) pulls in no test file by itself; when the search finds no `test_*.py` there, <none | all of `-m integration`> runs» (выбрать одно). Добавить «a helper in `tests/` stands for the unit files in `tests/` only».
  Passes: 3/3 (вывод grep в LADO `fc131c7`: `tests/ui/conftest.py`, `tests/integration/conftest.py`, `tests/integration/fake_provider.py`)

- **F3.7** [medium] `skills/lado-checks/SKILL.md:27`
  > | Code that drives processes, tmux, git, hooks, `lado mcp`, the server process or a provider (`runtime.py`, `hooks.py`, `mcp_server.py`, `tmux.py`, `state.py`, `loop.py`, `server/`, `providers/`) | also the integration tests: those the search below finds, or all of them (`uv run pytest -m integration`) when it finds none |

  Сама строка не менялась, но из-за нового `:47-49` integration-тесты идут «only through the row for processes, tmux, hooks, `lado mcp` and providers». Перечень строки не полный. Поиск связывает `runs.py` с `tests/integration/test_flow_runs.py`, а `runs.py` (в нём `git worktree add`) в перечне нет. То же для `terminal.py`, `agent_env.py`, `kits.py`, `gitcache.py`.

  Разработчик пропустит эти тесты, они упадут только на merge, и `red` вернёт run в `implement`, `review` и `merge_ok`. Пересказ строки в `:48` уже расходится с ней: в нём нет «git» и «the server process».
  Fix: в `:27` вместо перечня — «any module the search ties to an integration test»; в `:48` сослаться на строку, а не пересказывать её.
  Passes: full pass, confirmed (`grep … runs` в LADO → `tests/integration/test_flow_runs.py`)

- **F3.8** [medium] `skills/lado-checks/SKILL.md:68` (states `implement`, `review`)
  > they need passed: a skipped provider is not green; say why (environment, "When a check

  Провайдер, у которого CLI не установлен или не залогинен, пропускается (`tests/live/conftest.py:85-88`, `pytest.skip(f"{CLASSES[name].command} not installed")`). Разработчик называет причину «environment», но не сказано, с каким статусом он отчитывается, а Done шага `implement` требует, чтобы проверки были зелёными.

  Ревьюер освобождён только от одного случая: «or name their absence as an Important finding, unless the report says the human declined Claude's run» (`:86-87`). Исключение `:84` («a check that cannot run for an environment cause is no finding») относится к проверкам, которые ревьюер запускает сам, а live-тесты он не запускает. Получается Important, затем `changes` разработчику, который не может починить окружение, и так до `max_visits`. Это known hole 1 для live-тестов.
  Fix: в `:69` — «the developer reports DONE_WITH_CONCERNS naming it»; в `:87` — «…or the report names a provider skipped for an environment cause: then it is a concern for `merge_ok`, not a finding».
  Passes: 3/3

- **F3.9** [medium] `skills/lado-checks/SKILL.md:66`
  > On a no, run the other providers' live tests; the developer's report says "the human

  `:62-63` говорит: «A change to one provider runs only that provider's». Если изменение касается только `providers/claude.py`, то при «нет» разработчик гоняет Kilo и OpenCode, которые изменение не проверяют. Вместе с «a skipped provider is not green» (`:68`) пропуск бесплатной модели блокирует правку только Claude.

  Кроме того, `make test-live` берёт один `PROVIDER` (`Makefile:28`, `-k $(PROVIDER)`). Без `PROVIDER` снова идёт Claude, а `:57` требует запускать live «through `make test-live`». Агенты поступят по-разному.
  Fix: «On a no, run the live tests of the other providers the change needs, if any, one `make test-live PROVIDER=<name>` each».
  Passes: 3/3 (импакт: проходы дали medium, low, medium; взят medium)

- **F3.10** [medium] `skills/lado-checks/SKILL.md:114` (state `merge`, обе flow)
  > in the note, only for code or test: the task goes back to `implement`. For flaky, rerun

  Шаг 3 сначала велит классифицировать сбой по «When a check fails», а там flaky — это «passes on rerun with no change. Rerun up to 2 times» (`:134`). Узнать, что сбой flaky, можно только после перезапуска. Поэтому один супервизор вернёт сбой по таймингу как `red`, другой перезапустит один тест дважды, третий перезапустит весь `make check`.

  «red again, it is code or test» к тому же пропускает классификацию: повторный красный прогон может быть и environment.
  Fix: «If red, rerun the whole `make check` once before classifying: green → flaky (its BACKLOG.md entry, go on); red again → classify the first error as below».
  Passes: 2/3

- **F3.11** [low] `skills/lado-checks/SKILL.md:115` (state `merge`)
  > the whole `make check` once: green, go on and add a BACKLOG.md entry for the flake on

  Запись о flake коммитится после зелёного прогона. Done (`:121-122`, «main is at the run's branch and the check of step 3 was green on it») тогда относится к коммиту, на котором проверка не шла. Одни супервизоры запустят 4-минутный `make check` в третий раз, другие нет.
  Fix: «…(a commit of BACKLOG.md alone after the green check needs no new check)».
  Passes: 3/3

### 4. Independent verification

Не изменилось: `review` (`max_visits: 3`), затем гейт `merge_ok` с `needs: [implement]`, затем полный `make check` в `merge`.

### 5. Contradictions

- **F5.8** [medium] `skills/lado-checks/SKILL.md:47` (states `implement`, `review`)
  > UI tests run only when a path matches the `web/`, `src/lado/server/` row, integration tests

  Предложение задумано как ограничение для попаданий в conftest, но написано как общее правило. Тогда оно противоречит строке `:26` («each with its layer's `-m`») и изменённой `:33` («integration and UI hits still run with their `-m`»).

  Пример: поиск для `tmux.py` находит `tests/ui/test_terminal_panel.py`, для `kits.py` — `tests/ui/test_kits_page.py` и `tests/integration/test_agent_kits.py`. Ни `tmux.py`, ни `kits.py` не подходят под строку `web/`/`server/`, а `kits.py` — и под строку процессов. Разработчик и ревьюер выберут разные строки, отсюда находка «missing check» и лишний круг.
  Fix: «…does not pull in its whole layer by itself; test files the search finds there still run with their `-m`».
  Passes: 3/3 (grep в LADO для `tmux` и `kits`)

- **F5.9** [low] `agents/supervisor.md:32`
  > 5. A merged run cleans up its workers, worktree and branch. `flow_cancel` keeps them, and

  «them» включает и workers. Но `flow_cancel` их завершает: «Cancel an open flow run: its workers are finished, its worktree and branch are kept for you to merge or remove» (`src/lado/mcp_server.py:252-253`). В `lado-checks:171` сказано верно: «the run's worktree and branch are kept».
  Fix: «`flow_cancel` finishes its workers and keeps its worktree and branch, and ends a run only on the human's decision» (слов не прибавится больше чем на два; при пороге 1000 для лида запас есть).
  Passes: 3/3

- **F5.10** [low] `skills/lado-checks/SKILL.md:160`
  > the tier it belongs to (P0–P3, as BACKLOG.md's header says), with its `Size:` and `Why

  В шаблоне записи прямо над этой строкой (`:151-157`) строки `Size:` / `Why here:` нет. В BACKLOG.md LADO она стоит «under its title». Агент, который копирует шаблон, её пропустит или поставит в другое место.
  Fix: добавить в шаблон под заголовком `Size: <S|M|L>. Why here: <reason>.`
  Passes: 2/3

### 6. Duplication

- **F6.2** [low] `BLUEPRINT.md:36`
  > providers' live tests run and the human decides whether to release. *Source:* triage of

  В R9 человек решает после прогона других провайдеров, и нигде не сказано, что они должны быть зелёными. В скилле порядок обратный и с условием: «the human, having declined Claude's run, decided to release: then the other providers' live tests run and must be green» (`lado-checks:92-93`). Копии уже расходятся. План говорил сверить R9 с итоговым текстом скилла.
  Fix: выровнять R9 со скиллом: «…the other providers' live tests must be green, and the human decides whether to release».
  Passes: 2/3

### 7. When to call the human

В изменённом тексте нарушений нет. Платный прогон: «gets the human's yes first, every time». Удаление ветки отменённого run'а: «removes them only on their yes» (`lado-checks:172`). Ещё одна находка — в тексте, который diff не трогал (F7.1, «Missed earlier»).

### 8. Loops on a later visit

Не изменилось: `review` и `architecture` ограничены `max_visits: 3` и указывают себя в `needs`, `implement` тоже. Риск петли из нового правила о skipped-провайдере — F3.8.

### 9. Concision and why

Нарушений нет. Новые правила несут причину («live tests run serially», «a skipped provider is not green; say why»). Роли developer и supervisor — по 800 слов, сокращения не потеряли правил (см. «Cut rules»).

### 10. Skill descriptions

Описания и списки скиллов не менялись. Описание `lado-checks` говорит, что в скилле и когда его брать, и чем он отличается от соседей.

### 11. Provider neutrality

Имён инструментов CLI нет. `PROVIDER` — переменная Makefile LADO (`Makefile:27`: `PROVIDER=claude|kilo|opencode picks one`).

### 12. Safety and scope

В изменённом тексте нарушений нет. Удаление worktree и ветки — «only on their yes», платный прогон — с «да» человека. Ещё одна находка — в тексте, который diff не трогал (F12.1, «Missed earlier»).

## Known holes

| Known hole | Finding, or how the kit handles it |
|---|---|
| 1. Red check sent back with no environment cause considered | Merge закрыт: «For environment, tell the human what is missing and leave the step open» (`lado-checks:116-117`). Открыто для live-тестов: провайдер, пропущенный из-за окружения, ревьюер считает Important — **F3.8**. Что делает супервизор с сообщением ревьюера об окружении — **F7.1** (Missed earlier). |
| 2. Work outside a flow, merge without a gate | Закрыто для кода: «Any code goes through `fix` or `feature`, so every merge into main has a review and the human's `merge_ok`» (`agents/supervisor.md:62-63`). Разработчик вне run'а больше не коммитит BACKLOG.md (`agents/developer.md:65-66`). Коммит релиза — **F12.1** (Missed earlier). |
| 3. Path outside the run's worktree | Закрыто. Лог «outside the tree», макеты и бриф — абсолютные пути, правка `ui.md` — «on the run's branch». |
| 4. Verdict without a severity threshold | Закрыто: «The answer is Yes when no Critical or Important finding is open» (`agents/reviewer.md:82-83`). Связка critique↔макеты решена (`agents/reviewer.md:65`). Пропуск live — F3.8. |
| 5. Dependency skill that writes or asks where its role must not | Закрыто: `tdd` — «without asking the human» (`agents/developer.md:35`); `frontend-design` — «do not redesign or ask the human for a look»; вопросы из `receiving-code-review` идут через «a question for the human goes to the supervisor» (`agents/developer.md:100-101`); `critique-*` только оценивают. |

## Not traced

Нет. Каждая роль, шаг, гейт и скилл есть в трассировке BLUEPRINT.md §3, жёлтая мера обоснована в §4. Расхождение R9 со скиллом — F6.2.

## Previous findings

Находки отчёта `kit-reports/lado-dev-0.10.0-2026-10-06.md`, проверенные по пунктам плана:

| Находка | Статус | Доказательство |
|---|---|---|
| F3.1 live при «нет» на Claude, выбор PROVIDER | RESOLVED | `:62-63` «A change to one provider runs only that provider's (`PROVIDER=<name>`)»; `:66-67` «On a no, run the other providers' live tests; the developer's report says "the human declined Claude's run"». Остаток — F3.9. |
| F3.2 порог, область `make test`, `tests/conftest.py` | RESOLVED | `:33` «more than 10 unit test files for \| `make test` (all unit tests) in place of the unit hits»; `:45` «`tests/conftest.py` stands for the unit files in `tests/` only». Хелпер `tests/agent_helpers.py` — в F3.3. |
| F3.3 conftest UI/integration при поиске по пакету | STILL OPEN | UI исправлен (`:47`). Integration через хелпер `tests/integration/fake_provider.py` по-прежнему тянет весь слой или ничего — см. F3.3 выше. Проходы: STILL OPEN 2/3. |
| F3.4 skipped-провайдер не зелёный | RESOLVED | `:68` «a skipped provider is not green». Новая петля — F3.8. |
| F3.5 flaky в шаге merge | RESOLVED | `:114-116` «For flaky, rerun the whole `make check` once». Остатки — F3.10, F3.11. |
| F3.6 перезапуск после `red` | RESOLVED | `:78-79` «also the failing checks its note quotes (a test with its `-m`, or the make target)» |
| F5.1 макеты важнее critique | RESOLVED | `agents/reviewer.md:65` «Where a skill disagrees with `docs/design/ui.md`, the approved mockups or AGENTS.md, those win:» |
| F5.2 релиз после отказа от Claude | RESOLVED | `:91-92` «or the human, having declined Claude's run, decided to release». Расхождение с R9 — F6.2. |
| F5.3 `-n auto` и live | RESOLVED | `:55-57` «add `-n auto` to every unit, integration and UI pytest command … Live tests run serially, through `make test-live`.» (совпадает с `Makefile:28` `-n0`) |
| F5.4 кто действует в «When a check fails» | RESOLVED | `:126` «Only the developer fixes; a reviewer reports the class, the merge step acts as its step 3» |
| F6.1 R9 устарел | RESOLVED | `BLUEPRINT.md:33` «A release needs green CI and green live tests on the release commit»; строка 0.10.1 в журнале. Новое расхождение — F6.2. |
| F5.5 уровни P0–P3 | RESOLVED | `:159-160` «Add it at the end of the tier it belongs to (P0–P3, as BACKLOG.md's header says)». Шаблон — F5.10. |
| F5.6 BACKLOG разработчика вне run'а | RESOLVED | `agents/developer.md:65-66` «outside a run, list them under **Found on the way** in your report: the supervisor records them.» |
| F5.7 ветка после `flow_cancel` | RESOLVED | `:171-172` «it tells the human that the run's worktree and branch are kept, and removes them only on their yes». Формулировка в роли — F5.9. |
| F2.1 кто правит `docs/design/ui.md` | RESOLVED | `agents/supervisor.md:54-55` «what to change in `docs/design/ui.md`; the developer makes that change on the run's branch.» |

Fixed / not fixed автора сверен с файлами. Автор пишет, что F3.3 исправлен, но его интеграционная половина остаётся (см. выше). Остальное совпадает.

## Cut rules

| Removed rule (file:line at base) | Where it is now |
|---|---|
| «add a BACKLOG.md entry on your branch» (`agents/developer.md:64`) | `agents/developer.md:64-65` «add a BACKLOG.md entry … committed with your work» (в run'е коммит на ветку run'а) |
| «Done when each has an entry, committed with your work» (`agents/developer.md:65-66`) | Done шагов `implement`: «each **Found on the way** item has its BACKLOG.md entry (your role, section 4)» (`flows/feature.yaml:71-72`, `flows/fix.yaml:17-18`) |
| «the rest goes to BACKLOG.md (section 4)» (`agents/developer.md:99`) | `agents/developer.md:63-66` (§4) |
| «A run that ends cleans up its workers, worktree and branch» (`agents/supervisor.md:32`) | `:32` «A merged run cleans up…». Отменённый run — `lado-checks:171-172` (заменено намеренно, F5.7) |
| «`flow_cancel` ends a run only when the human decided to drop it» (`agents/supervisor.md:32-33`) | `:32-33` «ends a run only on the human's decision»; также `:79` «Cancel the run only on the human's decision» |
| «A UI design starts from `docs/design/ui.md` and updates it» (`agents/supervisor.md:51`) | `:54-55` «what to change in `docs/design/ui.md`; the developer makes that change on the run's branch» (заменено намеренно, F2.1) |
| «the developer builds to them, the reviewer checks against them» (`agents/supervisor.md:54-55`) | `agents/developer.md:45-46` «Build to the approved mockups the design names»; `agents/reviewer.md:61-62` «the approved mockups the design note names» |
| «On a no, the developer's report says the live check did not run» (`lado-checks:59`) | `:66-67`, заменено намеренно (F3.1) |
| «unless the report says the human declined the paid run: then it is a concern» (`lado-checks:77-78`) | `:87-88` «declined Claude's run: then Claude's absence is a concern for `merge_ok`» |
| «If the human declines Claude's paid run, run the other providers' live tests, and the human decides whether to release» (`lado-checks:81-83`) | `:91-93`. Порядок и условие «must be green» поменялись — F6.2 |
| «Add it at the end of the file» (`lado-checks:144`) | `:159-160` «at the end of the tier it belongs to», `merge=union` сохранён (заменено намеренно, F5.5) |
| «since the run's branch is removed with it» (`lado-checks:155`) | `:171-172`, заменено намеренно (F5.7) |

Потерянных правил нет.

## Missed earlier

Находки полного прохода в тексте, который diff не трогал. Вердикт они не блокируют и в stop rule не учитываются.

- **F12.1** [medium] `agents/supervisor.md:70`
  > Release only when the human asks, with the checks `lado-checks` names for a release; push

  Для релиза нужен коммит с новой версией: «Release: `uv version <X.Y.Z>`, commit, then push tag `vX.Y.Z`» (LADO `AGENTS.md:66`). По киту `pyproject.toml` и `uv.lock` — это код («Any other path is code», `lado-checks:32`), а «Any code goes through `fix` or `feature`» (`:62-63`). Значит, супервизор либо коммитит на main без ревью, вопреки своему правилу, либо открывает `fix` ради одной строки версии.

  `lado-checks:91` требует «green CI … on the release commit», а CI LADO запускается только на push в main или на PR (`.github/workflows/ci.yml`: `push: branches: [main]`). Push тега сразу публикует в PyPI (`release.yml`: `tags: ["v*"]`). Нигде не сказано: сначала запушить main, дождаться зелёного CI, потом тег. Это шаг, который уходит с машины, и агенты сделают его по-разному.
  Fix: в §4: «the version bump is the one direct commit on main the human's release request allows; push main, wait for green CI on it, then push the tag» (или «a `fix` run for the bump»).
  Passes: full pass, confirmed (AGENTS.md:66, ci.yml, release.yml в LADO `fc131c7`)

- **F7.1** [medium] `agents/reviewer.md:86`
  > fails", says: an environment failure is no finding; send the supervisor what is missing

  Ревьюер оставляет `review` открытым и шлёт супервизору сообщение об окружении. У супервизора правило только для других статусов: «A worker's NEEDS_CONTEXT or BLOCKED message leaves its step open» (`agents/supervisor.md:77`). Не сказано отнести недостающее человеку и попросить ревьюера перезапустить проверку, когда окружение починят. Run стоит на `review`, пока кто-нибудь не заметит.
  Fix: в `agents/supervisor.md:77`: «A worker's NEEDS_CONTEXT, BLOCKED or environment message leaves its step open: …» — заменить, а не добавить, роль у порога.
  Passes: full pass, confirmed

## Left by the plan

- Похожие абзацы (бюджет жёлтый: review 92%, implement 79%, вступления architect/reviewer 77%). Причина человека: две flow задуманы так (R2, R6), шаги различаются источником AC, каждый `do` читается сам. Записано в BLUEPRINT.md §4.

## Found on the way

- `[lado]` `flow_cancel` завершает workers, а `finish_worker` по описанию «only for workers started outside a run» в ките. После отмены worktree и ветку run'а остаётся удалять через git (`runs.py:656-657`: «with no workers left, remove them with git»). Кит этого не говорит. Стоит решить, нужен ли LADO инструмент для удаления ветки отменённого run'а.
