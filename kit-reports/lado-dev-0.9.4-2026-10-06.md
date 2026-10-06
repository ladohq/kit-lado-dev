# Kit report: lado-dev 0.9.4

- Дата: 2026-10-06
- Кит: worktree запуска `improve/lado-dev` (`.lado/worktrees/kit-lado-dev/improve-lado-dev`). Это путь из задачи шага, ветка `lado/kit-lado-dev/improve-lado-dev`.
- Коммит: `58ea55f`
- Оценивал: критик kit-builder (слои a и b)
- Режим: полная оценка, шаг `assess`. Отчёт 0.9.3 не подходит для повторного использования: в 0.9.4 текст кита изменился (`58ea55f`), поэтому кит оценён заново. Прошлые отчёты при этом не читались.
- Проходы: три независимых суб-агента. Каждый получил рубрику, `lado-kit-format`, папку кита и папки всех 21 зависимых скиллов из кэша LADO (все найдены). Отчёты из `kit-reports/` им не давали.
  - Проход 1 начинал с kit.yaml и ролей.
  - Проход 2 шёл по flows шаг за шагом, с трассировкой нот.
  - Проход 3 начинал с `lado-checks`, затем шли зависимые скиллы и роли.
  - Критик свёл три списка и проверил каждую цитату через `grep -nF`.
  - Отброшено 8 находок, которые дал только один проход. Одна однопроходная находка оставлена как подтверждённая (F6.3).
  - Повтор правила RESOLVED/STILL OPEN в роли и в `do` нашли все три прохода. Он не засчитан, как и в отчётах 0.9.2 и 0.9.3: `lado-kit-format` («Notes and `needs`») и критерий 8 требуют, чтобы `do` повторяющегося шага говорил о повторном визите.
- Факты о репозитории LADO критик проверил сам в `~/IdeaProjects/lado`, коммит `f1ab799`:
  - `Makefile`, `pyproject.toml`: `check: lint test-js web browser`, затем `uv run pytest -m 'not live' -n auto`. Это unit-, integration- и UI-тесты в одном прогоне.
  - addopts исключает `integration`, `live` и `ui`.
  - `.github/workflows/ci.yml`.

Находки — кандидаты, которые взвешивает человек. Это не оценка «прошёл / не прошёл».

## Card

| Слой | Результат |
|---|---|
| a. `lado kits check` | OK, предупреждений нет |
| a. Бюджет | жёлтый. Все числовые меры зелёные. Жёлтые только из-за трёх пар похожих абзацев: `review` в feature/fix — 92%, `implement` в feature/fix — 78%, вступления architect/reviewer — 77%. `developer.md` (797) и `supervisor.md` (798) упираются в порог 800. |
| b. Рубрика | 11 находок: 2 high, 7 medium, 2 low. Без находок 7 из 12 критериев (2, 4, 7, 8, 9, 11, 12). |
| Охват | полная оценка всего кита: 9 файлов (kit.yaml, README.md, 4 роли, 2 flow, `lado-checks`) и 21 зависимый скилл |

### `lado kits check .`

```
lado-dev: OK (3 agents, 71 skills, 4 packs and 2 flows)
```

### Budget script (exit status 0)

```
# Complexity budget: lado-dev 0.9.4

| Measure | Where | Value | Green / yellow up to | Zone |
|---|---|---|---|---|
| Worker roles (not supervisor) | kit | 3 | 3 / 5 | green |
| Work steps in a flow | flows/feature.yaml | 5 | 5 / 8 | green |
| Work steps in a flow | flows/fix.yaml | 3 | 5 / 8 | green |
| Gates in a flow | flows/feature.yaml | 2 | 2 / 3 | green |
| Gates in a flow | flows/fix.yaml | 1 | 2 / 3 | green |
| Words in a role prompt | agents/architect.md | 576 | 800 / 1500 | green |
| Words in a role prompt | agents/developer.md | 797 | 800 / 1500 | green |
| Words in a role prompt | agents/reviewer.md | 766 | 800 / 1500 | green |
| Words in the lead's prompt | agents/supervisor.md | 798 | 1000 / 1500 | green |
| Own skills | kit | 1 | 5 / 10 | green |
| MCP servers | kit | 0 | 2 / 4 | green |

## Similar paragraphs (one rule, one place; 55% similar or more)

- yellow: 92% similar: flows/feature.yaml: state "review": "Review the run's branch against main, read-only, as your role describes: the ..." ~ flows/fix.yaml: state "review": "Review the run's branch against main, read-only, as your role describes: the ..."
- yellow: 78% similar: flows/feature.yaml: state "implement": "Implement the design (the note from design, as the human approved it) in the ..." ~ flows/fix.yaml: state "implement": "Implement the task in the run's worktree, test-first, as your role describes...."
- yellow: 77% similar: agents/architect.md: "Most reviews come as a step of a flow run (a message from `lado`): the design..." ~ agents/reviewer.md: "Most reviews come as a step of a flow run (a message from `lado`): the step s..."

Overall: yellow
```

## Fix first

1. В разделе 4 ревьюера классифицировать красную проверку по `lado-checks` («When a check fails»). При классе environment сообщить супервизору и не отвечать `changes` (F3.1).
2. Задать ревьюеру порог серьёзности для «Yes»: открытые Critical/Important блокируют, Minor — нет (F3.2).
3. Обновить таблицу `lado-checks`: добавить `make test-ui` и `make web`, исправить «It runs all four above», дать целевые формы для `-m integration` и `-m ui` (F3.3).
4. Убрать из таблицы «`make test` … After any change; fastest feedback» (F5.1).
5. Ограничить или убрать `brainstorming` у супервизора и `domain-modeling` у архитектора: скиллы, которые пишут и коммитят, стоят на ролях, которым это нельзя (F5.2, F1.1).

## Findings

### 1. Role boundaries

- **F1.1** [medium] `agents/architect.md:38`
  > `domain-modeling` for module boundaries and names.
  Архитектор «change[s] no files» (`:11`) и не пишет человеку (`:60-61`). При этом `domain-modeling` велит создать `CONTEXT.md` и `docs/adr/` («If no `CONTEXT.md` exists, create one when the first term is resolved. If no `docs/adr/` exists, create it when the first ADR is needed.») и спорить с «the user». Сам скилл говорит, что простое чтение словаря — не его задача. Это known hole 5.
  Fix: убрать `domain-modeling` из `skills:` архитектора, так как `codebase-design` уже даёт словарь. Другой вариант — добавить «только его вопросы; термин или ADR, который стоит записать, идёт в ревью как находка или вопрос человеку». Проходы разошлись в критерии (5 и 1); взят 1, он ближе к исправлению. Passes: 3/3

### 2. Handoffs between steps

Проверены все переходы. В первом визите `implement` получает ревью архитектора через гейт: «The architect's review comes with the human's answer in the previous step's note on the first visit» (`flows/feature.yaml:62-63`). `merge` получает одобряющее ревью через `merge_ok`. Все нужные ноты пишутся целиком: «note_body the whole design, ACs included» (`flows/feature.yaml:35-36`).

### 3. Done and outcomes

- **F3.1** [high] `agents/reviewer.md:86` (state `review`, обе flow)
  > while there are uncommitted changes or a red check.
  Любой красный `make check` даёт «No», то есть `changes`, и задача уходит в `implement`. Сбой окружения сюда тоже попадает: npm без сети в `web`, нет Chromium для `browser`, tmux, uv. Ревьюер не классифицирует сбой. Различение environment / code есть только в шаге merge (`lado-checks:70-71`). В итоге разработчику возвращается «баг», которого нет в работе, и тратится один из трёх визитов `review`. С тех пор как в `make check` вошли `web` и UI-тесты, такие сбои стали вероятнее. Это known hole 1. Проходы дали medium/medium/high; взят high: run теряет визит и может упереться в `max_visits`.
  Fix: в разделе 4 ревьюера написать: «Красную проверку классифицируй по `lado-checks`, "When a check fails". Класс environment — не находка: сообщи супервизору через `send_message`, что отсутствует, и оставь шаг открытым». Passes: 3/3

- **F3.2** [high] `agents/reviewer.md:85` (state `review`, обе flow)
  > End with **Ready to merge: Yes | No | With fixes**, and one line why. The answer is not Yes
  У находок есть серьёзность (Critical / Important / Minor, `:75`), но нигде не сказано, какая из них исключает «Yes». Обе flow переводят любой «With fixes» в `changes`. Ветка с одними Minor-находками в одном прогоне уходит на доработку, в другом проходит. Цикл может дойти до `max_visits: 3`. У архитектора порог есть: «`approved` when no Critical or Important finding» (`flows/feature.yaml:49`). Шкала `critique-*` (minor / major issue) тоже ни на что не отображена. Это known hole 4. Проходы дали medium/medium/high; взят high.
  Fix: «Yes, когда нет открытых Critical и Important; Minor (и `minor issue` критики) не блокируют, `major issue` считается Important». Passes: 3/3

- **F3.3** [medium] `skills/lado-checks/SKILL.md:19`
  > | The change is ready | `make check` | Developer: last, before you report done, except a change only to non-code paths (below), which needs `make lint` only. Reviewer and merge step: as the rules below say. It runs all four above. |
  `make check` запускает `lint test-js web browser`, затем pytest unit+integration+ui. «All four above» неверно. Строк для `make test-ui` и `make web` в таблице нет. Целевая форма из `:29` (`uv run pytest tests/test_x.py -k`) из-за addopts молча отбирает ноль integration- и ui-тестов. Разработчик UI (`agents/developer.md:45-47`) остаётся без команды для своего слоя. Красный тест в этом слое «проходит» пустым прогоном.
  Fix: добавить строки для `make test-ui` (после изменений UI и его тестов) и `make web` (после изменений в `web/`). Описать, что на самом деле запускает `make check`. В `:29` дать формы `-m integration` и `-m ui`. Passes: 3/3

### 4. Independent verification

feature: `architecture`, затем `design_ok` (`needs: [design]`), затем `review`, затем `merge_ok` (`needs: [implement]`), затем `merge`. fix: `review`, затем `merge_ok`, затем `merge`. Ни один исход не обходит ревью: `outcomes: {done: review}`.

### 5. Contradictions

- **F5.1** [medium] `skills/lado-checks/SKILL.md:16`
  > | Pure logic works | `make test` | After any change; fastest feedback. |
  Противоречит `:29` («While working, run only the tests next to your change») и `:27` («run each one once»). К тому же это неверно по факту: `make test` идёт 3 мин 41 с, `make lint` — 3,5 с. Агент, который следует таблице, гоняет почти четырёхминутный набор после каждого шага TDD. Это прямая причина проблемы из задачи.
  Fix: в столбце «When» строк `:16-17` написать «входит в `make check`; отдельно — только чтобы сузить падение». Убрать «fastest feedback». Passes: 3/3

- **F5.2** [medium] `agents/supervisor.md:43`
  > work around), and recommend one. Use `brainstorming` when the shape is unclear.
  В архитектурной ветке `brainstorming` пишет «save to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md` and commit», то есть коммит на main из checkout супервизора без ревью. Это против `:63`: «every merge into main has a review». Дальше скилл вызывает `writing-plans`, которому во flow нет места. Его «Only one question per message» противоречит раундам `grilling` одним сообщением (`flows/feature.yaml:26`). Это known hole 5.
  Fix: «`brainstorming` — только для вариантов и их цены; дизайн — это нота шага, без spec-файла, коммита и `writing-plans`; вопросы — раундами, как в шаге design». Другой вариант — убрать скилл: раздел 2 уже требует 2–3 варианта. Passes: 3/3

- **F5.3** [medium] `agents/developer.md:33` (state `implement`)
  > reason, then the code that makes it pass, then the next. Test at the seams the brief names,
  `tdd` говорит: «Before writing any test, write down the seams under test and confirm them with the user. No test is written at an unconfirmed seam.» Ни бриф (`agents/supervisor.md:23-25`), ни шаг design (`flows/feature.yaml:18`) не обязаны называть seams. А человеку разработчик не пишет (`agents/developer.md:100`). От прогона к прогону будет то NEEDS_CONTEXT, то выбранные без подтверждения seams. Это known hole 5. Проходы дали критерии 2, 5 и 5; взят 5.
  Fix: добавить «the seams under test» в состав брифа и в шаг design 3. В роль разработчика добавить: «seams из брифа считаются подтверждёнными; если их нет — выбери по слоям AGENTS.md и перечисли в отчёте». Passes: 3/3

### 6. Duplication

- **F6.1** [low] `agents/supervisor.md:80`
  > - A paid `make test-live PROVIDER=claude` needs the human's yes every time, also inside a
  Одно правило стоит в четырёх местах: здесь, в `skills/lado-checks/SKILL.md:20`, в `:35-36` и в `agents/developer.md:69-70`. По плану задачи из ролей уходят упоминания проверок, так что копии в ролях надо свести к `lado-checks`.
  Fix: оставить правило в `lado-checks`, в ролях — только указатель. Passes: 2/3

- **F6.2** [low] `flows/fix.yaml:17` (state `implement`; тот же текст в `flows/feature.yaml:70-75`)
  > Done when every AC about behaviour is covered by a test you saw fail and then pass,
  Условие готовности и формат отчёта повторены слово в слово в обеих flow. Это повтор `agents/developer.md:37` и раздела 5. Бюджет видит это как пару с 78% сходства. Отсылка «the final check of your role's section 5» привязывает обе flow к тексту роли о проверках, который по плану задачи уходит.
  Fix: оставить формат отчёта в разделе 5 разработчика, а в `do` — указатель. Финальную проверку назвать через `lado-checks`. Passes: 2/3

- **F6.3** [medium] `agents/developer.md:68`
  > 1. After your last change run `make check`, or only `make lint` when you changed only
  Правило выбора финальной проверки пересказано в роли, и две копии уже разошлись. В `skills/lado-checks/SKILL.md:31-32` есть случай «a change to a kit … needs `lado kits check`», в роли его нет. Плюс роль требует полный `make check` от разработчика, хотя по замыслу человека он нужен только в merge. Проверено: обе строки на месте (`grep -nF`), третьего случая в `developer.md` нет.
  Fix: заменить на «запусти финальную проверку, которую `lado-checks` называет для твоего изменения». Passes: 1/3, confirmed

### 7. When to call the human

Гейты принадлежат человеку (`agents/supervisor.md:29-31`). Вопросы, NEEDS_CONTEXT и BLOCKED идут супервизору (`agents/developer.md:76-78, 100-101`, `agents/architect.md:57-61`). «A step at its visit limit opens a gate» (`agents/supervisor.md:83`).

### 8. Loops on a later visit

`design`, `architecture`, `implement` и `review` указывают себя в `needs`. Ревью стоят на `max_visits: 3` и помечают RESOLVED / STILL OPEN. `implement` при каждом визите пишет отчёт целиком и знает, где взять находки: «the previous step's note says what to fix».

### 9. Concision and why

No-op'ов в ролях не найдено. Неочевидные правила несут причину: «a rebase rewrites commits the review relies on» (`agents/developer.md:26`), «git's "Already up to date" is only a hint» (`skills/lado-checks/SKILL.md:46-47`). Устаревшая таблица разобрана в F3.3 и F5.1.

### 10. Skill descriptions

- **F10.1** [medium] `kit.yaml:16`
  > folders: [skills/engineering, skills/productivity]
  Каждый внешний скилл нужно объявлять его собственной папкой (`lado-kit-format`). Здесь и в `kit.yaml:22` (`folders: [interaction-design/skills, visual-critique/skills]`) объявлены родительские папки. В `kit.yaml:13` superpowers объявлен без `folders`. Итог: 71 установленный скилл, из них используется 21. Часть лишних тянет агентов против ролей:
  - `subagent-driven-development` против «without sub-agents» (`agents/developer.md:97`);
  - `finishing-a-development-branch` (merge/push/PR);
  - `writing-plans`, на который передаёт `brainstorming`;
  - `improve-codebase-architecture`, на который ссылается `diagnosing-bugs`.

  Fix: перечислить папку каждого используемого скилла, например `skills/engineering/tdd`, `skills/verification-before-completion`, `interaction-design/skills/state-machine`. Passes: 3/3

Описание `lado-checks` говорит, что в скилле и когда его брать, и отделяет его от `verification-before-completion` / `diagnosing-bugs`.

### 11. Provider neutrality

Названий инструментов CLI, моделей и конфигов нет. `PROVIDER=claude` и CLAUDE.md — собственные имена репозитория LADO. Используются только инструменты LADO (`flow_advance`, `spawn_worker`, `lado answer`).

### 12. Safety and scope

Merge в main — локальный `--ff-only` после гейта `merge_ok`. Тег пушится только по просьбе человека: «Release only when the human asks» (`agents/supervisor.md:70`). Платный test-live требует «да» человека. `flow_cancel` и `discard=True` — только по решению человека. Обход без ревью через `brainstorming` разобран в F5.2.

## Known holes

| Known hole | Finding, or how the kit handles it |
|---|---|
| 1. Red check sent back with no environment cause considered | В `review` дыра открыта: **F3.1**. В merge закрыта: «for environment, tell the human what is missing and leave the step open» (`skills/lado-checks/SKILL.md:71`). У разработчика тоже: «do not change code to work around it» (`:90`). |
| 2. Work outside a flow, merge without a gate | Закрыто: «Outside a flow goes only read-only work … Any code goes through `fix` or `feature`, so every merge into main has a review and the human's `merge_ok`» (`agents/supervisor.md:61-64`). Остаток — коммит spec через `brainstorming`, см. **F5.2**. |
| 3. Path outside the run's worktree | Закрыто. Бриф и макеты передаются абсолютным путём (`agents/supervisor.md:26, 54`). «Work only inside your worktree (in a run, the run's worktree)» (`agents/developer.md:21`). Логи — «outside the tree» (`skills/lado-checks/SKILL.md:54`). |
| 4. Verdict without a severity threshold | У архитектора порог есть (`flows/feature.yaml:49`). У ревьюера нет: **F3.2**. |
| 5. Dependency skill that writes or asks where its role must not | **F1.1** (`domain-modeling` у read-only архитектора), **F5.2** (`brainstorming` коммитит у супервизора), **F5.3** (`tdd` подтверждает seams с пользователем). Остальное закрыто. `frontend-design` перекрыт фразой «do not redesign or ask the human for a look» (`agents/developer.md:45`). Вопросы `diagnosing-bugs` и `receiving-code-review` идут супервизору (`agents/developer.md:100`). |

## Questions for the human

1. **CI и UI-тесты.** В задаче сказано, что CI не запускает UI-тесты и сборку web. Но в `~/IdeaProjects/lado/.github/workflows/ci.yml` (последнее изменение — 2026-10-03) есть job `ui`: `make dist` (включает `make web`) и `uv run pytest -m ui -n auto`. Job `check` запускает lint, unit и integration, job `js` — `test-js`. Значит, CI покрывает всё, что входит в `make check`. Рекомендация: уточнить. Если CI действительно зелёный на каждом push в main, полный `make check` на main перед релизом дублирует CI. Может хватить правила «зелёный CI на коммите релиза + `make test-live`», которое уже есть в `lado-checks:51-52`. Если CI в этом репозитории не запускается (нет раннеров или он выключен), решение человека остаётся в силе.
2. **Целевые тесты.** Unit-тесты целиком идут почти 4 минуты, и их тоже нужно сужать. Рекомендация: в `lado-checks` дать агенту явное правило, как связать изменённый файл с тестами. Например, `src/lado/X.py` соответствует `tests/test_X.py` и тестам, которые импортируют X (`grep -l`). Для `web/` — `make web`, для UI — `pytest -m ui -k …`. И сказать, что делать, когда соответствия нет: запустить `make test`. Без этого «тесты рядом с изменением» каждый агент будет понимать по-своему.
