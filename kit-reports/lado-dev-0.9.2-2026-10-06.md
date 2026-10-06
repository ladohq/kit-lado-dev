# Kit report: lado-dev 0.9.2

- Дата: 2026-10-06
- Кит: /Users/aleksejkolesnikov/IdeaProjects/kit-lado-dev. Путь был дан в задаче (коммит `8e049ee`). Worktree запуска стоит на том же коммите.
- Оценивал: критик kit-builder (слои a и b).
- Проходы: три независимых суб-агента. Каждый получил только рубрику, `lado-kit-format` и папку кита; прошлый отчёт им не показывали.
  - Проход 1 начинал с kit.yaml и ролей.
  - Проход 2 шёл по flows шаг за шагом, с трассировкой нот.
  - Проход 3 начинал со скилла `lado-checks`, затем шли роли.
  - Критик свёл три списка и проверил каждую цитату через `grep -nF`. Прошлые находки он проверял сам.
  - Отброшено 9 находок, которые нашёл только один проход. Одна из них подтверждена по git и, вероятно, стоит внимания. Это потерянное уточнение «with the log outside the tree». В старом `merge` (0.9.1, `flows/feature.yaml:116-117`) оно было. В `lado-checks:52` после переноса процедуры его нет: лог `make check` теперь может лечь в worktree запуска.
  - Три находки два прохода отнесли к одному месту и одной причине, но к разным критериям. Они засчитаны под критерием большинства, второй критерий указан при каждой.

Находки — кандидаты, которые взвешивает человек. Это не оценка «прошёл / не прошёл».

## Card

| Слой | Результат |
|---|---|
| a. `lado kits check` | OK, предупреждений нет |
| a. Бюджет | зелёный. Все меры зелёные, дубликатов абзацев нет. `developer.md` (799) и `supervisor.md` (799) упираются в порог 800: любое добавление в них, например для исправления F12.1 или F2.1, сделает их жёлтыми. |
| b. Рубрика | 7 находок: 1 high, 2 medium, 4 low. Без находок 8 из 12 критериев (1, 3, 4, 7, 8, 9, 10, 11). |

### `lado kits check /Users/aleksejkolesnikov/IdeaProjects/kit-lado-dev`

```
lado-dev: OK (3 agents, 71 skills, 4 packs and 2 flows)
```

### Budget script (exit status 0)

```
# Complexity budget: lado-dev 0.9.2

| Measure | Where | Value | Green / yellow up to | Zone |
|---|---|---|---|---|
| Worker roles (not supervisor) | kit | 3 | 3 / 5 | green |
| Work steps in a flow | flows/feature.yaml | 5 | 5 / 8 | green |
| Work steps in a flow | flows/fix.yaml | 3 | 5 / 8 | green |
| Gates in a flow | flows/feature.yaml | 2 | 2 / 3 | green |
| Gates in a flow | flows/fix.yaml | 1 | 2 / 3 | green |
| Words in a role prompt | agents/architect.md | 589 | 800 / 1500 | green |
| Words in a role prompt | agents/developer.md | 799 | 800 / 1500 | green |
| Words in a role prompt | agents/reviewer.md | 766 | 800 / 1500 | green |
| Words in a role prompt | agents/supervisor.md | 799 | 800 / 1500 | green |
| Own skills | kit | 1 | 5 / 10 | green |
| MCP servers | kit | 0 | 2 / 4 | green |

## Duplicate paragraphs (one rule, one place)

none

Overall: green
```

## Fix first

1. Работа разработчика вне flow не должна попадать в main без ревью и gate. Либо ограничить раздел «Outside a flow» работой только на чтение, либо отправлять код через `fix` (F12.1).
2. При отклонении `merge_ok` шаг `implement` должен знать, что чинить: добавить в список «что чинить» причину отказа человека (F2.1).
3. Свести правило о **Found on the way** отменённого запуска в одно место и закрыть потерю записей, если запуск отменён после `implement` (F5.1).

## Findings

### 1. Role boundaries

Находок нет. Права каждой роли названы явно:
- архитектор: «You change no files.» (`agents/architect.md:11`);
- ревьюер: «You change no files: no fixes, no commits on the branch you review.» (`agents/reviewer.md:19`);
- супервизор: «You do not write code; you design, delegate, decide and merge.» (`agents/supervisor.md:11-12`);
- разработчик: «Work only inside your worktree» (`agents/developer.md:21`).

Запись BACKLOG.md в `merge` закреплена за супервизором в `lado-checks:58-62`.

### 2. Handoffs between steps

- **F2.1** [medium] `flows/feature.yaml:68` (state `implement`; то же в `flows/fix.yaml:13`)
  > `      report is the note from implement, and the previous step's note says what to fix:`
  Переход `merge_ok: rejected → implement` (`flows/feature.yaml:102`, `flows/fix.yaml:46`) тоже ведёт в этот шаг. Тогда в ноте лежат ответ человека и одобряющее ревью. Причины в списке (review findings, конфликт с main, красный `make check`) этот случай не покрывают. Разработчик может решить, что чинить нечего, или взяться за Minor-находки ревью и пропустить причину отказа. Следующий `review` нужен только себе и `design`, ответа человека он не видит. Выполнение просьбы человека тогда проверяет только её автор.
  Fix: в обоих `implement` добавить в список «or the human's reason for rejecting the merge (quote it at the top of your report)». В `review` добавить «check a quoted rejection like an AC». Passes: 3/3 (два прохода отнесли это к критерию 2, один — к 8: «the fixing step not told where the findings to fix are»)

### 3. Done and outcomes

Находок нет. У каждого рабочего `do` есть условие готовности и условие для каждого исхода. В `merge` они заданы через `lado-checks:63-74` («report `conflict`», «report `red`», «Then report `merged`»). NEEDS_CONTEXT и BLOCKED явно оставляют шаг открытым (`agents/developer.md:77-79`), а супервизор знает, что с ними делать (`agents/supervisor.md:75-77`).

### 4. Independent verification

Находок нет.
- В `feature` перед `implement` стоят архитектор и gate `design_ok` с `needs: [design]`.
- В обоих flows перед `merge` стоят ревьюер и gate `merge_ok`. Теперь у него есть `needs: [implement]` (`flows/feature.yaml:100`, `flows/fix.yaml:44`), так что человек видит отчёт разработчика и одобряющее ревью.

Пробел при отказе человека описан в F2.1, обход ревью вне flow — в F12.1.

### 5. Contradictions

- **F5.1** [medium] `skills/lado-checks/SKILL.md:117` против `agents/supervisor.md:88`
  > `approved the branch in `merge`, and those of a run it cancels before `implement` on main.`
  > `- A bug, friction or debt you find, and the open **Found on the way** items of a run you`
  Скилл велит супервизору записывать на main пункты только того запуска, который отменён до `implement`. Роль говорит об «open» пунктах любого отменённого запуска, а куда их записывать, не говорит. После `implement` записи лежат только на ветке запуска. Ветку LADO удаляет, когда запуск заканчивается («A run that ends cleans up its workers, worktree and branch», `agents/supervisor.md:32`). Если запуск отменят после `implement`, эти записи тихо пропадут.
  Fix: в скилле написать «and every item of a run it cancels that has no entry on main yet, added on main». Роль сократить до указателя на `lado-checks`. Passes: 2/3 (один проход отнёс это к критерию 6)

- **F5.2** [low] `agents/supervisor.md:81`
  > `- If three review rounds pass without the open findings going down, take the question to`
  Правило ссылается на механизм, который считает другое. `max_visits: 3` считает визиты в `review`, в том числе после `conflict`/`red` на `merge`, а не то, уменьшились ли находки. На `architecture` правило не распространяется. Супервизор может написать человеку раньше, чем LADO откроет gate, или ждать застоя, который лимит не заметит.
  Fix: «When a review step reaches its visit limit, LADO opens a gate; tell the human in one line that it waits.» Отдельный триггер убрать. Passes: 2/3 (один проход отнёс это к критерию 8)

### 6. Duplication

- **F6.1** [low] `flows/feature.yaml:73` (state `implement`; то же в `flows/fix.yaml:18` и `agents/developer.md:69`)
  > `      After your last change run `make check`, or only `make lint` for a change only to`
  Правило финальной проверки записано в четырёх местах: таблица `lado-checks:19`, роль разработчика (раздел 5.1) и оба `implement`. Правило записей BACKLOG тоже повторено: `agents/developer.md:62-65` и оба `implement` (`flows/feature.yaml:71-72`, `flows/fix.yaml:16-17`). Копии уже расходятся.
  - Роль: «when you changed only non-code paths», flows: «for a change only to non-code paths».
  - Скобка «(the design's and the architect's)» в `feature` не упоминает пункты ревью `changes`, а роль и `lado-checks:116` их упоминают.
  Fix: оставить правило в `lado-checks` и в роли. В `do` оставить только относящееся к шагу (условие готовности) и короткий указатель. Passes: 3/3

- **F6.2** [low] `flows/fix.yaml:50` (state `merge`) против `flows/feature.yaml:106-110`
  > `      Follow `lado-checks`, section "Merging a run's branch", with the approving review in`
  Оба `merge` — один и тот же шаг, но написан двумя разными текстами. В `fix` перечислены подшаги («record its **Found on the way** items, merge main in, check, fast-forward main»). В `feature` пересказано, что LADO делает после `merged` («LADO then ends the run, closes its workers and removes its worktree and branch», строка 109), хотя это уже есть в `agents/supervisor.md:32`. Оба заново формулируют условия исходов из `lado-checks`.
  Fix: один короткий текст в обоих flows: «Merge as `lado-checks`, "Merging a run's branch", says; the approving review is in the note. Report `merged`, `conflict` or `red` as it says.» Passes: 2/3

- **F6.3** [low] `agents/architect.md:63` против `flows/feature.yaml:48-52` (state `architecture`)
  > `Verdict: `approved` when no Critical or Important finding and no question for the human is`
  Правило вердикта и содержимое note_body («your review only … LADO gives the design») записаны и в роли (`agents/architect.md:63-67`), и в `do` шага `architecture`. Так же устроены «Re-review» ревьюера и строки `review`. Повтор RESOLVED/STILL OPEN в `do` требует `lado-kit-format`, поэтому находкой он не считается. Остальное — повтор.
  Fix: в `do` оставить связь исходов с вердиктом. В роли — формат отчёта, без второй копии условия. Passes: 2/3

### 7. When to call the human

Находок нет. Решения человека доходят до него тремя путями:
- через «Questions for the human» архитектора и раунды `grilling` в `design`;
- через «a question for the human goes to the supervisor» (`agents/developer.md:100-101`, `agents/reviewer.md:92-93`);
- через правило супервизора о NEEDS_CONTEXT/BLOCKED (`agents/supervisor.md:75-77`).

Несходящиеся петли ограничены `max_visits: 3` (оговорка о формулировке — в F5.2).

### 8. Loops on a later visit

Находок нет.
- `design`, `architecture`, `implement` и `review` нужны сами себе (`implement` — с 0.9.2) и говорят, что меняется при повторном визите.
- `implement` пишет отчёт целиком: «starts with every AC, written out in full». Разработчик на повторном визите тоже: «the full report as in 5, the AC list first» (`agents/developer.md:93`).
- Петли ограничены `max_visits: 3` или gate `merge_ok`.

Пробел на пути отказа человека описан в F2.1.

### 9. Concision and why

Находок, которые нашли два прохода, нет. Неочевидные правила объяснены:
- запрет rebase: «since a rebase rewrites commits the review relies on» (`agents/developer.md:25-26`);
- запрет pipe: «`make check | tail` hides a red exit status» (`lado-checks:51-52`);
- `merge=union` для BACKLOG.md (`lado-checks:109-110`).

Повторы описаны в разделе 6.

### 10. Skill descriptions

Находок, которые нашли два прохода, нет. Описание `lado-checks` говорит, что в нём (в том числе «how the merge step merges a run's branch») и когда его брать. Каждый скилл из `skills:` роли вызывается в её тексте или в её шагах. `requesting-code-review` и `finishing-a-development-branch` удалены.

### 11. Provider neutrality

Находок нет. Нет имён инструментов CLI, моделей и конфигов. Действия названы нейтрально: «your editing tools (edit, write)», «If you cannot view images, say so». Путей к файлам кита нет. `PROVIDER=claude|kilo` — параметр `make test-live` самого LADO, а не зависимость кита.

### 12. Safety and scope

- **F12.1** [high] `agents/supervisor.md:61-62`
  > `with `spawn_worker(role=...)` and a self-contained brief: a developer for code, an`
  > `architect or reviewer for a look. End it with `finish_worker(name)` when its work is merged`
  Раздел «Outside a flow» отдаёт код разработчику вне запуска и предполагает, что его работа будет «merged». Ни ревью, ни gate, ни процедура слияния для этого случая не заданы: «Merging a run's branch» в `lado-checks` описывает только запуски. Это противоречит строке 19 («For every task from the human, start a run with `flow_start`»). Правило «Every branch, however small, is reviewed before it is merged» (строка 80) ревью требует, а согласия человека — нет. Супервизор может влить код в main без «да» человека. В 0.9.1 было похоже («not merged into your current branch»), но там хотя бы говорилось, куда идёт слияние. После сокращения не осталось и этого.
  Fix: вне flow — только работа на чтение («a question, an investigation, a look at a branch»), код идёт через `fix` или `feature`. «when its work is merged» заменить на «when it has reported». Passes: 2/3

## Previous findings

Отчёт `kit-reports/lado-dev-0.9.1-2026-10-06.md`, 13 находок. Закрыты все 13.

- **F2.1** RESOLVED. `merge_ok` получил `needs: [implement]`: `flows/feature.yaml:100`, `flows/fix.yaml:44`. Новый пробел на пути `rejected` — в F2.1 этого отчёта.
- **F5.1** RESOLVED. Таблица и правило совпадают. `lado-checks:19`: «Developer: last, before you report done, except a change only to non-code paths (below), which needs `make lint` only.» Оба `implement` говорят «or only `make lint` for a change only to non-code paths».
- **F5.2** RESOLVED. `agents/developer.md:25`: «Rebase your branch on `main` only before your first commit on it; after that, merge main as the step says, since a rebase rewrites commits the review relies on.»
- **F5.3** RESOLVED. `agents/developer.md:93`: «the body is the full report as in 5, the AC list first,».
- **F6.1** RESOLVED. Бюджет: «Duplicate paragraphs … none». Оба `merge` ссылаются на «Merging a run's branch», правило пропуска осталось только в `lado-checks:43-48`. Ревьюер ссылается: «Run the checks on the branch as `lado-checks`, "Run as little as proves the claim"» (`agents/reviewer.md:31`). Остаточное расхождение двух `merge` — в F6.2.
- **F6.2** RESOLVED. `agents/supervisor.md:88-89`: «… go to BACKLOG.md as `lado-checks` says.» Новое расхождение роли и скилла по отменённым запускам — в F5.1.
- **F6.3** RESOLVED. `flows/feature.yaml:18`: «3. Write a short design as your role says, with what changes and where, how it is».
- **F7.1** RESOLVED. `agents/supervisor.md:75-77`: «A worker's NEEDS_CONTEXT or BLOCKED message leaves its step open: answer it from the brief or the design with `send_message`, or ask the human and pass the answer on. Cancel the run only on the human's decision.»
- **F9.1** RESOLVED. `grep -n "task tracker is connected" skills/lado-checks/SKILL.md` ничего не находит.
- **F10.1** RESOLVED. `grep -rn requesting-code-review agents` ничего не находит.
- **F10.2** RESOLVED. `grep -rn finishing-a-development agents` ничего не находит.
- **F12.1** RESOLVED. `agents/supervisor.md:78-79`: «A paid `make test-live PROVIDER=claude` needs the human's yes every time, also inside a run.» В `lado-checks:20`: «ask the human, through the supervisor, every time».
- **F12.2** RESOLVED. `agents/supervisor.md:64`: «Use `discard=True` only for work the human decided to throw away.»

## Questions for the human

1. Может ли супервизор вне flow поручать разработчику код, который потом вливается, или всякий код идёт через `fix`/`feature` (F12.1)? Рекомендация: только через flow. Тогда у каждого слияния в main есть ревью и gate `merge_ok`, а «Outside a flow» остаётся для работы на чтение.
2. Что делать с **Found on the way** запуска, отменённого после `implement` (F5.1)? Рекомендация: супервизор переносит на main каждый пункт, у которого там ещё нет записи, на любом шаге отмены.
