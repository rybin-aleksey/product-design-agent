# Шаблоны выходных данных

`ds/` означает `workspace/product/ds/`.

| Файл | Фаза | Назначение |
|---|---|---|
| `tasks/<TASK-ID>/analysis/grow-ui-kit-draft.md` | 2 | Черновик md-описаний для апрува |
| `ds/components.md` | 4 | Дописаны записи новых компонентов с Node ID |

---

## Черновик описаний (фаза 2)

```markdown
# grow_ui_kit — черновик описаний

**Дата:** <ISO-дата>
**Задача:** <TASK-ID>
**Набор:** <Table, Tabs, Toast>

### <ComponentName>

**Назначение:** одна строка, что это в продукте.

**Анатомия:**
- <слой 1> (например, Header)
- <слой 2> (например, Row)
- <слой 3> (например, Cell)

**Варианты (2–3):**
- <Property>: <val1> | <val2>

**Состояния:** default (остальные — отдельной задачей на расширение матрицы)

**Вложенные компоненты ДС:** <Button (`1:234`) / нет>

**Отступы:** <padding: space-3 / space-4; itemSpacing: space-2>

**Привязки:**
- fills → <surface-* / bg-*>
- strokes → <border-*>
- text color → <text-*>
- typography → <DS/Body/sm, DS/Label/xs>
- radius → <radius-*>

**Блокеры:** <нет / не хватает токена X или стиля Y>
```

---

## Запись в `ds/components.md` (фаза 4)

```markdown
### Table
- **Назначение:** табличный список с заголовками и строками
- **Варианты:** Density: compact | default
- **Node ID:** `1:512`
```

Шапку файла обновить:

```markdown
**Кол-во наборов компонентов:** <N>
**Расширено:** <ISO-дата>, grow_ui_kit (<TASK-ID>): <Table, Tabs, Toast>
```

---

## Отчёт аудита (фаза 4.2)

```text
Аудит «UI Kit — extended» против правил «Foundation»:

Table       ✓
Tabs        ✗ paddingLeft=14 без токена (Tab) → привязать к space-3 (12) или space-4 (16)
Toast       ✗ fill прибит у Icon → привязать к text-on-action

Расхождений: 2. Исправляю через use_figma и повторяю аудит.
```

---

## Финальный отчёт

```text
UI-кит расширен:
- ds/components.md — дописано N записей (всего наборов: M)
- Секция «UI Kit — extended» в Figma — N компонентов плиткой
- Все привязаны к Semantic Variables и TextStyles ДС, прибитых hex нет
- Аудит на консистентность пройден

Черновик описаний: workspace/product/tasks/<TASK-ID>/analysis/grow-ui-kit-draft.md

Не подтверждено / открыто: <например, ручная проверка Inspect не делалась>

Что дальше:
- Расширение матрицы вариантов (hover / focus / disabled / loading) — скилл design-system-variants (component_variants)
- Сборка экранов из UI-кита — скилл final-screens (final_screens), по ds/CONTRACT.md: новые компоненты заводятся через те же правила, ad-hoc на экранах запрещён
```
