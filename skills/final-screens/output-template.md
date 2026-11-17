# Шаблоны выходных данных

`ds/` — `workspace/product/ds/`, `task/` — `workspace/product/tasks/<TASK-ID>/`.

| Файл | Фаза | Назначение |
|---|---|---|
| `task/artifacts/design/screens/<name>.md` | 2, 4 | Composition map экрана |
| `task/artifacts/design/screens/_index.md` | 5 | Экраны, собранные в задаче |
| `task/analysis/final-screens-extract-<name>.md` | 2.5 | Брифы выносимых компонентов |
| `ds/components.md` | 2.5, 5 | Новые компоненты, `used in` |
| `ds/foundation.md` | 2.5 | Новые Semantic (alias) |

---

## Нет компонента под секцию (фаза 1.3)

```text
Не нашёл компонента под секцию «<имя>». Варианты:
1. Добавить компонент в ДС (рекомендую; при 3+ копиях — в этом прогоне, иначе design-system-grow)
2. Взять ближайший — <кандидат>, потеряв часть смысла: <что именно>
3. Оставить плейсхолдер wireframe

Что выбираем?
```

## Анализ достаточности (фаза 1.4)

```text
Анализ wireframe <ScreenName>:

[1] СЫРАЯ СУЩНОСТЬ: «<карточка>» содержит только <поля>. Предлагаю раскрыть до:
    <icon + name + status-badge + меню → ключевой атрибут → chips → footer>
    Принять? (да / убрать часть / оставить как в wireframe)

[2] ПУСТАЯ ЗОНА: заполнено ~<N>%. Предлагаю добавить: <секции>.
    Принять всё / выбрать часть / оставить?

[3] EDGE CASES не заложены. Предлагаю: <empty / loading / error / overflow — где>.
    Заложить? (да / нет / только …)

[4] МАЛО ДЕЙСТВИЙ: <предложение>. Принять?

[5] ГОЛЫЙ ТЕКСТ: <FAQ / ссылки> → <chevron / accordion>. Принять?

[6] СПИСОК С ОДНИМ ЭЛЕМЕНТОМ → <N> элементов с разными статусами. Принять?
```

Пункты без проблем не показываются.

## Повторяющиеся блоки (фаза 1.5)

```text
Повторяющиеся блоки:

[A] <BlockName> — <N> копий в <секции>. Вынести в UI-кит?
    Варианты: <Property: a | b | c>
    Где ещё пригодится: <экран / нет данных>
    Выносим? (да / нет / отложить)
```

---

## Composition map (фаза 2)

```markdown
# Screen: <ScreenName>

**Задача:** <TASK-ID>
**Дата:** <ISO-дата>
**Wireframe Node ID:** `<id>`
**Final Screen Node ID:** <заполняется в фазе 4>
**Ширина:** 1440, высота HUG

## Решения брифа
- Анализ достаточности: <принято / отклонено по пунктам>
- Вынос в UI-кит: <ServerCard — да; StepItem — локально, кандидат на вынос>
- Edge cases: <empty, loading / не закладываем по решению пользователя>

## Composition

### Header
- Auto Layout: horizontal, space-between, padding space-4 / space-6, gap space-6
- Sizing: FILL × HUG
- Фон: surface-default; тень: —
- Содержимое (слева направо):
  - Placeholder/Logo 120×32 (заменить руками)
  - Navbar, Node ID `1:222`
  - IconButton, Node ID `1:444`, Type=ghost, Size=md, иконка Lucide / bell 16

### Stats Row
- Auto Layout: horizontal, gap space-4; Sizing: FILL × HUG
- 3 × StatCard, Node ID `5:120`, каждая FILL × HUG, тень shadow-sm
  - Поля: label (DS/Body/sm, text-muted) · value (DS/Heading/2xl) · trend badge (Badge, Variant=success)
- Демо: «Баланс / 12 400 ₽ / +4%», … (иллюстративно)

### Servers List
- Auto Layout: vertical, gap space-3; Sizing: FILL × HUG
- 3 × ServerCard, Node ID `5:101`, Status=running | stopped | deploying
  - Мета: OS — иконка + текст; Region — флаг + текст; Specs — 3 chips; Uptime — muted + Lucide / clock 12

## Edge cases

### Empty state (Servers List)
- Сосед списка в той же секции, visible: false
- Иллюстрация-плейсхолдер, DS/Heading/lg «Серверов пока нет», Button Type=primary «Создать сервер»

### Loading (Servers List)
- 3 × ServerCard State=loading (если есть в ДС; иначе — скелет-фреймы surface-subtle), visible: false

## Использовано
- Компоненты: <список с Node ID>
- Semantic: surface-default, surface-subtle, text-default, text-muted, border-default
- Отступы: space-3, space-4, space-6
- TextStyles: DS/Heading/2xl, DS/Body/sm
- EffectStyles: shadow-sm
- Иконки (Lucide): bell, clock, chevron-right, copy
- Плейсхолдеры: Placeholder/Logo, Placeholder/Illustration

## Демо-контент
Все цифры, имена и статусы — иллюстративные. <Что взято из контекста продукта / что нужно заменить реальными данными>.
```

---

## Запись в `ds/components.md` (фаза 5)

```markdown
### Card
- **Назначение:** контейнер контента
- **Варианты:** Type: default | interactive
- **Node ID:** `1:777`
- **Used in:** TASK-001/screens/dashboard, TASK-001/screens/file-detail
```

## Индекс экранов (фаза 5)

```markdown
# Screens — <TASK-ID>

- [dashboard.md](dashboard.md) — главный экран после входа; Final Screen `1:bbb`
- [file-detail.md](file-detail.md) — карточка файла с превью и историей версий; Final Screen `1:ccc`
```

---

## Отчёт: single

```text
Экран <ScreenName> собран:
- Final Screen Node ID: <id>
- Компонентов ДС: <N> (<список>)
- Вынесено в UI-кит в этом прогоне: <ServerCard — ComponentSet 5:101, Status (4) / нет>
- Новые Semantic: <text-trend-positive → success-500 / нет>
- Edge cases: <empty, loading — visible: false / нет>
- Плейсхолдеры на замену руками: <Logo, Illustration>
- Демо-контент: иллюстративный, <что заменить реальными данными>

Файлы:
- tasks/<TASK-ID>/artifacts/design/screens/<name>.md, _index.md
- ds/components.md (used in<, новые компоненты>)<, ds/foundation.md>

Проверено по факту: инстансы вместо фреймов, привязки цветов, Auto Layout и sizing, TextStyles, тени, иконки SVG.

Дальше: контрольный аудит экранов — screens_audit (скилл screens-audit).
```

## Отчёт: bulk

```text
Прогон по wireframes завершён.

Собрано:
- Dashboard — 9 компонентов, 2 edge cases, 1 плейсхолдер; +3 компонента в UI-кит, +2 Semantic
- Settings — 6 компонентов, edge cases не закладывались (решение пользователя)

Пропущено:
- Onboarding — «соберу позже руками»
- File Detail — уже собран, пересборка не запрашивалась

UI-кит: +<N> компонентов; Foundation: +<K> Semantic (все alias).
Обновлены: _index.md, ds/components.md, ds/foundation.md.
```
