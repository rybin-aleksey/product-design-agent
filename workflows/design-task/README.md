# Design task

Workflow для выполнения отдельной дизайн-задачи продукта.

## Как выбрать ветку

1. Определи задачу: `workspace/product/tasks/<TASK-ID>/`.
2. Проверь, есть ли у продукта ДС: если существует `workspace/product/ds/CONTRACT.md`, прочитай контракт до начала работы с интерфейсом.
3. Выбери ветку по запросу.

| Запрос | Ветка |
|---|---|
| «Создай / поставь дизайн-систему с нуля», «заведи токены и UI-кит», у продукта нет ДС | [Создание дизайн-системы с 0](#ветка-создание-дизайн-системы-с-0) |
| «Подключи / проиндексируй нашу ДС», «работаем по корпоративной ДС», «взяли ДС из Community», ДС в Figma уже есть | [Подключение существующей дизайн-системы](#ветка-подключение-существующей-дизайн-системы) |

Все файлы дизайн-системы в обеих ветках лежат в `workspace/product/ds/` — это общая папка продукта, а не папка задачи. Рабочие заметки прогона — в `workspace/product/tasks/<TASK-ID>/analysis/`.

## Ветка: Создание дизайн-системы с 0

Запускает директиву ds_baseline из скилла [skills/design-system-from-0](../../skills/design-system-from-0/SKILL.md).

**Условие входа:** у продукта нет действующей ДС, то есть нет `workspace/product/ds/CONTRACT.md`. Если ДС уже есть, ветка не запускается: расширение UI-кита и ребрендинг — отдельные задачи. Если ДС есть в Figma, но не проиндексирована — ветка «Подключение существующей дизайн-системы».

**Шаги:**

1. Прочитай [SKILL.md](../../skills/design-system-from-0/SKILL.md), затем [methodology.md](../../skills/design-system-from-0/methodology.md).
2. **Бриф.** Задай пять вопросов и сохрани `ds/brand-brief.md`.
3. **Foundation.** Собери `ds/foundation.md` и **дождись апрува**. До апрува никаких MCP-вызовов.
4. **Figma.** Получи ссылку на Figma-файл продукта, создай Variables и секцию «Foundation» с 7 базовыми компонентами, проверь результат по факту и запиши `ds/components.md`.
5. **Контракт.** Создай `ds/CONTRACT.md`. Корневые `CLAUDE.md` и `AGENTS.md` не меняй.
6. Сверься с [checklist.md](../../skills/design-system-from-0/checklist.md) и выдай финальный отчёт по [output-template.md](../../skills/design-system-from-0/output-template.md).

**Результат:** `workspace/product/ds/` (brand-brief, foundation, components, CONTRACT), а в Figma коллекции Primitive и Semantic и секция «Foundation».

**Ветка завершена**, только когда пройдена фаза 4 (контракт). Без неё ДС остаётся декорацией.

## Ветка: Подключение существующей дизайн-системы

Запускает директиву ds_scan из скилла [skills/design-system-scan](../../skills/design-system-scan/SKILL.md).

**Условие входа:** ДС уже собрана в Figma (своя, корпоративная или дублированная из Community), а её индекса в `workspace/product/ds/` нет или он устарел. Если ДС в Figma нет — ветка «Создание дизайн-системы с 0».

**Шаги:**

1. Прочитай [SKILL.md](../../skills/design-system-scan/SKILL.md), затем [methodology.md](../../skills/design-system-scan/methodology.md).
2. **Контекст.** Задай четыре вопроса, добейся ссылки именно на страницу Components и сохрани `ds/source.md`.
3. **Foundation.** Одним read-only вызовом прочитай Variables, собери `ds/foundation.md`, покажи пользователю и уточни неясные токены.
4. **Components.** Прочитай только верхний уровень страницы Components, собери `ds/components.md` с Node ID и пройди self-check.
5. **Показ.** Выдай сводку с замечаниями.
6. **Контракт.** Создай `ds/CONTRACT.md` с правилами под источник. Корневые `CLAUDE.md` и `AGENTS.md` не меняй.
7. Сверься с [checklist.md](../../skills/design-system-scan/checklist.md) и выдай финальный отчёт по [output-template.md](../../skills/design-system-scan/output-template.md).

**Результат:** `workspace/product/ds/` (source, foundation, components, CONTRACT). В Figma ничего не меняется.

**Ветка завершена**, только когда пройдена фаза 5 (контракт). Без неё индекс остаётся справочником.
