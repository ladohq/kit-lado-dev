# Kit report: lado-dev 0.9.3

- Дата: 2026-10-06
- Кит: /Users/aleksejkolesnikov/IdeaProjects/kit-lado-dev. Путь был дан в задаче (коммит `18c02aa`). Worktree запуска стоит на том же коммите.
- Оценивал: критик kit-builder (слои a и b).
- Проходы: три независимых суб-агента. Каждый получил только рубрику, `lado-kit-format` и папку кита; прошлые отчёты им не показывали.
  - Проход 1 начинал с kit.yaml и ролей.
  - Проход 2 шёл по flows шаг за шагом, с трассировкой нот.
  - Проход 3 начинал со скилла `lado-checks`, затем шли роли.
  - Критик свёл три списка и проверил каждую цитату через `grep -nF`. Прошлые находки и правки 0.9.3 (`git diff 484021b 18c02aa`) он проверял сам.
  - Отброшено 9 находок, которые нашёл только один проход. Две из них, по мнению критика, стоят внимания:
    - Общее правило про лог (`lado-checks:52`, «write it to a log (`make check > <log> 2>&1`)») не говорит, где лежит лог. «outside the tree» вернулось только в шаг merge (`:67`). Лог разработчика или ревьюера в общем worktree запуска — незакоммиченный файл, а ревьюер не может дать Yes «while there are uncommitted changes» (`agents/reviewer.md:86`).
    - Никому не поручено запускать `make test-live`. Проход 3 заметил, что раздел 5 разработчика требует только `make check`, хотя `lado-checks:33-36` полагается на «the developer's report shows it green». Проход 1 заметил, что ревьюер не знает, что делать, пока ждёт «да» человека на платный прогон. Таблица (`:20`) и список (`:33`) к тому же по-разному говорят, когда его запускать.
  - Повтор правила RESOLVED/STILL OPEN в роли и в `do` нашли все три прохода. Он не засчитан: как и в отчёте 0.9.2, `lado-kit-format` («Notes and `needs`») и критерий 8 требуют, чтобы `do` повторяющегося шага говорил о повторном визите.

Находки — кандидаты, которые взвешивает человек. Это не оценка «прошёл / не прошёл».

## Card

| Слой | Результат |
|---|---|
| a. `lado kits check` | OK, предупреждений нет |
| a. Бюджет | зелёный. Все меры зелёные, дубликатов абзацев нет. `developer.md` (799) и `supervisor.md` (797) по-прежнему упираются в порог 800. |
| b. Рубрика | 7 находок: 0 high, 4 medium, 3 low. Без находок 6 из 12 критериев (1, 4, 5, 7, 8, 11). Все 7 находок отчёта 0.9.2 и потеря «log outside the tree» закрыты. |

### `lado kits check /Users/aleksejkolesnikov/IdeaProjects/kit-lado-dev`

```
lado-dev: OK (3 agents, 71 skills, 4 packs and 2 flows)
```

### Budget script (exit status 0)

```
# Complexity budget: lado-dev 0.9.3

| Measure | Where | Value | Green / yellow up to | Zone |
|---|---|---|---|---|
| Worker roles (not supervisor) | kit | 3 | 3 / 5 | green |
| Work steps in a flow | flows/feature.yaml | 5 | 5 / 8 | green |
| Work steps in a flow | flows/fix.yaml | 3 | 5 / 8 | green |
| Gates in a flow | flows/feature.yaml | 2 | 2 / 3 | green |
| Gates in a flow | flows/fix.yaml | 1 | 2 / 3 | green |
| Words in a role prompt | agents/architect.md | 576 | 800 / 1500 | green |
| Words in a role prompt | agents/developer.md | 799 | 800 / 1500 | green |
| Words in a role prompt | agents/reviewer.md | 766 | 800 / 1500 | green |
| Words in a role prompt | agents/supervisor.md | 797 | 800 / 1500 | green |
| Own skills | kit | 1 | 5 / 10 | green |
| MCP servers | kit | 0 | 2 / 4 | green |

## Duplicate paragraphs (one rule, one place)

none

Overall: green
```

## Fix first

1. Публиковать макеты только с «да» человека. Иначе давать локальный путь (F12.1).
2. В шаге merge перед `red` классифицировать сбой. Ошибки окружения и flaky-тесты не отправлять в `implement` (F3.1).
3. Дизайн должен называть абсолютный путь к макетам, иначе разработчик и ревьюер в worktree запуска их не найдут (F2.1).
4. Согласовать условие готовности в роли разработчика со словами шагов: «every AC about behaviour» (F6.1).

## Findings

### 1. Role boundaries

Находок нет. Права каждой роли названы явно:
- архитектор: «You change no files.» (`agents/architect.md:11`);
- ревьюер: «You change no files: no fixes, no commits on the branch you review.» (`agents/reviewer.md:19`);
- супервизор: «You do not write code; you design, delegate, decide and merge.» (`agents/supervisor.md:11-12`);
- разработчик: «Work only inside your worktree» (`agents/developer.md:21`).

У записей BACKLOG.md один владелец на каждом шаге (`lado-checks:112-119`).

### 2. Handoffs between steps

- **F2.1** [medium] `agents/supervisor.md:52`
  > `static, self-contained HTML pages, never committed, for example in `.lado/mockups/<run>/`;`
  Макеты не закоммичены и лежат в дереве супервизора, а путь в примере относительный. Дизайн только «names where the approved mockups are». Разработчик и ревьюер работают в worktree запуска, где `.lado/mockups/...` нет («Work only inside your worktree», `agents/developer.md:21`). Разработчик может строить UI без макетов, ревьюер — проверять без них. Для брифов это уже решено: «the task names its absolute path» (`agents/supervisor.md:26`).
  Fix: «…for example in `.lado/mockups/<run>/`; the design names their absolute path.» Passes: 2/3

### 3. Done and outcomes

- **F3.1** [medium] `skills/lado-checks/SKILL.md:67-68` (state `merge`, оба flows)
  > `   lets you skip it, never piped, with the log outside the tree. If it is red, report`
  > `   `red`, with the failing output in the note.`
  Исход `red` выбирается при любом красном `make check`, в том числе flaky или окружения. Тот же скилл велит перезапускать flaky до двух раз (`:83-85`) и не менять код из-за окружения (`:86-87`). Супервизор отправит проблему tmux или логина в `implement`. Там разработчик её не починит, а визит `review` (`max_visits: 3`) уйдёт впустую.
  Fix: в шаге 3: «classify the failure as "When a check fails" says; report `red` only for code or test; for environment, tell the human and leave the step open». Passes: 2/3

### 4. Independent verification

Находок нет.
- В `feature` перед `implement` стоят архитектор и gate `design_ok` с `needs: [design]`.
- В обоих flows перед `merge` стоят ревьюер и gate `merge_ok` с `needs: [implement]`.
- Отказ человека на `merge_ok` теперь проверяет ревьюер: «and a quoted rejection of the merge like an AC» (`flows/feature.yaml:86`, `flows/fix.yaml:32`).
- Вне flow код больше не делается (`agents/supervisor.md:60-63`).

### 5. Contradictions

Находок нет. Проверено:
- правило отменённого run в роли и скилле: роль только указывает на `lado-checks` («go to BACKLOG.md as `lado-checks` says», `agents/supervisor.md:88-89`);
- «Outside a flow» против «For every task from the human, start a run» (`agents/supervisor.md:19`): теперь вне flow только работа на чтение;
- ссылки шагов на роли (`your role, section 4/5`) — разделы существуют и говорят то же;
- gates оставлены человеку («Never answer a gate or pretend to», `agents/supervisor.md:29-30`), все роли сообщают исход через `flow_advance`.

### 6. Duplication

- **F6.1** [medium] `agents/developer.md:38` против `flows/feature.yaml:70`, `flows/fix.yaml:17` (state `implement`)
  > `Done when every AC is covered by a test you saw fail and then pass.`
  Две копии условия готовности уже расходятся. Шаги говорят «Done when every AC about behaviour is covered by a test», роль — «every AC». Для AC о документации или конфиге разработчик, который следует роли, напишет бессмысленный тест (или проходящий по построению — это запрещает строка 37) или сообщит BLOCKED. Тот, кто следует шагу, — нет.
  Fix: в роли написать «every AC about behaviour», или убрать предложение из роли и оставить его в шагах. Passes: 2/3

- **F6.2** [low] `flows/feature.yaml:87` (state `review`; то же `flows/fix.yaml:33`)
  > `      Review the Architecture axis too, and list what is outside the change under`
  Ось Architecture (`agents/reviewer.md:47`) и правило **Found on the way** (`agents/reviewer.md:77-81`) уже есть в роли. Оба `review` повторяют их, не добавляя ничего, что относится к шагу.
  Fix: убрать это предложение из обоих `review`. Passes: 2/3

### 7. When to call the human

Находок нет. Вопросы архитектора идут через раунды `grilling` в `design`. Работники пишут супервизору («a question for the human goes to the supervisor», `agents/developer.md:100-101`, `agents/reviewer.md:92-93`). NEEDS_CONTEXT/BLOCKED оставляют шаг открытым (`agents/supervisor.md:76-78`). О лимите визитов: «A step at its visit limit opens a gate; tell the human as in 1.4.» (`agents/supervisor.md:82`).

### 8. Loops on a later visit

Находок нет.
- `design`, `architecture`, `implement` и `review` нужны сами себе и говорят, что меняется при повторном визите.
- `implement` пишет отчёт целиком («starts with every AC, written out in full»).
- Путь `merge_ok: rejected → implement` описан: «or the human's reason for rejecting the merge (quote it at the top of your report)» (`flows/feature.yaml:68-69`, `flows/fix.yaml:15-16`).
- Петли ограничены `max_visits: 3` или gate `merge_ok`.

### 9. Concision and why

- **F9.1** [low] `agents/developer.md:98`
  > `- Do the work yourself, without sub-agents.`
  Правило неочевидно, и агент CLI с суб-агентами его, скорее всего, нарушит, а причины у него нет.
  Fix: добавить причину, например «since the review and your report rely on one author for each change». Passes: 2/3

### 10. Skill descriptions

- **F10.1** [low] `skills/lado-checks/SKILL.md:3`
  > `description: Which LADO check proves which claim, how to read a failure, how the merge step merges a run's branch, and how to record a bug, friction or debt in BACKLOG.md and who records it. Use before saying work on the LADO repo is done, when a check fails, when you merge a run's branch, or when you find a LADO bug or debt.`
  Триггеры «before saying work … is done» и «when a check fails» совпадают с `verification-before-completion` и `diagnosing-bugs`, которые разработчик тоже держит. Чем этот скилл отличается от них, описание не говорит. Критик считает риск небольшим: роль сама зовёт их вместе (`agents/developer.md:69-71`).
  Fix: добавить «(which LADO command proves what; `verification-before-completion` and `diagnosing-bugs` hold the general discipline)». Passes: 2/3

### 11. Provider neutrality

Находок нет. Нет имён инструментов CLI, моделей и конфигов. Действия названы нейтрально: «your editing tools (edit, write)», «If you cannot view images, say so». Путей к файлам кита нет. `PROVIDER=claude|kilo` — параметр `make test-live` самого LADO.

### 12. Safety and scope

- **F12.1** [medium] `agents/supervisor.md:53`
  > `publish a page and give the link if your environment can. The design names where the`
  Супервизор публикует макеты за пределы машины по своему усмотрению («if your environment can»), без «да» человека. Публикация — одно из действий, которым критерий 12 требует gate или явное согласие. Строка есть с 0.9.1 (`c8afb8f`, тогда «If your environment can publish a page for the human to open, publish it»); прошлые отчёты её пропустили.
  Fix: «publish a page and give the link only when the human agrees; otherwise give the local path». Passes: 3/3

## Previous findings

Отчёт `kit-reports/lado-dev-0.9.2-2026-10-06.md`, 7 находок и одна потеря из отброшенных. Закрыты все.

- **F2.1** RESOLVED. `flows/feature.yaml:68-69`: «a red `make check`, or the human's reason for rejecting the merge (quote it at the top of your report).» То же в `flows/fix.yaml:15-16`. Ревьюер: «and a quoted rejection of the merge like an AC» (`flows/feature.yaml:86`, `flows/fix.yaml:32`).
- **F5.1** RESOLVED. `lado-checks:117-119`: «When it cancels a run, at any step, it adds on main every **Found on the way** item of the run that has no entry on main yet, since the run's branch is removed with it.» Роль теперь только указывает на скилл: «and the **Found on the way** items of a run you cancel, go to BACKLOG.md as `lado-checks` says» (`agents/supervisor.md:88-89`).
- **F5.2** RESOLVED. `agents/supervisor.md:82`: «A step at its visit limit opens a gate; tell the human as in 1.4.»
- **F6.1** RESOLVED. Оба `implement` теперь указывают на роль: «each **Found on the way** item has its BACKLOG.md entry (your role, section 4), the final check of your role's section 5 is green» (`flows/feature.yaml:71-72`, `flows/fix.yaml:18-19`). Своих копий правила проверки и записей BACKLOG у них больше нет. Расхождение «every AC» в роли — новая находка F6.1.
- **F6.2** RESOLVED. Оба `merge` одинаковы: «Follow `lado-checks`, "Merging a run's branch".» (`flows/feature.yaml:102`, `flows/fix.yaml:48`). Условия исходов и ноты — в `lado-checks:63-74`.
- **F6.3** RESOLVED. Условие вердикта осталось только в `do` (`flows/feature.yaml:49-50`). Роль: «In a run, report the outcome the step names with `flow_advance`» (`agents/architect.md:63`). Ревьюер наоборот: шаг указывает на роль («as your role, section 4, says», `flows/feature.yaml:92`).
- **F12.1** RESOLVED. `agents/supervisor.md:60-63`: «Outside a flow goes only read-only work (a question, an investigation, a look at a branch) … Any code goes through `fix` or `feature`, so every merge into main has a review and the human's `merge_ok`.»
- **Потеря «log outside the tree»** (отброшенная в 0.9.2) RESOLVED для шага merge: `lado-checks:67`: «lets you skip it, never piped, with the log outside the tree.» Общее правило `:52` места лога по-прежнему не называет. Это отброшено как находка одного прохода, см. «Проходы».

## Questions for the human

1. Можно ли супервизору публиковать макеты (например, приватной страницей) без вашего «да» на каждый раз (F12.1)? Рекомендация: публиковать только с согласия, по умолчанию давать локальный путь. Публикация уводит черновик с машины, а локального файла для решения о дизайне хватает.
2. Должен ли красный `make check` из-за окружения (tmux, логин CLI, сеть) в шаге merge возвращать run разработчику (F3.1)? Рекомендация: нет. Супервизор говорит вам о проблеме и оставляет шаг открытым, а в `implement` идут только сбои класса code/test.
