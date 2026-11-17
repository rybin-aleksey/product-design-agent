# Шаблоны выходных данных

`ds/` означает `workspace/product/ds/`.

| Файл | Фаза | Назначение |
|---|---|---|
| `tasks/<TASK-ID>/analysis/component-variants-<name>.md` | 3 | Черновик описания матрицы для апрува |
| `ds/components.md` | 5 | Обновлена строка вариантов у компонента |
| `ds/foundation.md` | 5 | Дописаны Semantic-токены (если потребовались) |

---

## Бриф (фаза 2)

```text
Сейчас у <ComponentName>: <currentMatrix> (<N> ячеек)

1. Варианты. Что добавить? (например: ghost; или пропускаем)
2. Состояния. Какие добавить? (например: hover, disabled, loading, focus; или пропускаем)
3. Размеры. Какие добавить? (например: sm, lg; или пропускаем)
```

---

## Предлагаемые токены (фаза 3.2)

```text
В ds/foundation.md (Semantic) и в Figma добавляются:
- surface-action-primary-hover → accent-700
- surface-action-secondary-hover → gray-100
- border-focus-ring → accent-500

Уже есть, не добавляем: border-error

Подтверждаешь?
```

---

## Черновик описания (фаза 3.3)

```markdown
# component_variants — <ComponentName>

**Дата:** <ISO-дата>
**Задача:** <TASK-ID>
**Node ID:** `<id>`
**Было:** <currentMatrix> (<N> ячеек)
**Станет:** <newMatrix> (<M> ячеек)
**Прогон:** <один / 1 из 2: варианты × состояния / 2 из 2: размеры>

## Ячейки

Существующие (не трогаем):
- Type=primary, State=default, Size=md
- …

Новые:
- Type=primary, State=hover, Size=md
- …

Исключены: <например, loading только у primary / нет>

## Состояния

| Состояние | Fill | Stroke | Прочее |
|---|---|---|---|
| hover | surface-action-<type>-hover | — | — |
| disabled | как default | — | opacity 0.4 |
| loading | как default | — | контент скрыт, плейсхолдер 16 px |
| focus | как default | border-focus-ring, 2px, outside | — |

## Размеры

| Размер | Padding X / Y | Gap | TextStyle | Плейсхолдер спиннера |
|---|---|---|---|---|
| sm | space-3 / space-1 | space-1 | DS/Body/sm | 12 |
| md | space-4 / space-2 | space-2 | DS/Body/sm Medium | 16 |
| lg | space-6 / space-3 | space-2 | DS/Body/base | 20 |
```

---

## Запись в `ds/components.md` (фаза 5.1)

```markdown
### Button
- **Назначение:** основная кнопка действий
- **Варианты:** Type: primary | secondary | ghost; State: default | hover | disabled | loading; Size: sm | md | lg
- **Node ID:** `1:234`

> Расширено <ISO-дата> (<TASK-ID>): состояния hover / disabled / loading, размеры sm / lg.
> Новые Semantic: surface-action-primary-hover, surface-action-secondary-hover, border-focus-ring.
```

## Запись в `ds/foundation.md` (фаза 5.2)

```markdown
### Поверхности (расширено <ISO-дата>)
- surface-action-primary-hover → accent-700
- surface-action-secondary-hover → gray-100

### Обводки (расширено <ISO-дата>)
- border-focus-ring → accent-500
```

---

## Отчёт: single

```text
Компонент <ComponentName> расширен:
- Матрица: <newMatrix>, <M> ячеек (было <N>)
- Новые Semantic в ds/foundation.md и Figma: <список / не потребовались>
- ds/components.md обновлён<, Node ID изменился: <old> → <new>>

Проверено по факту: оси в Properties, привязки fills/strokes, TextStyles, ячейка <имя>.
На ручную проверку: <например, смена accent-700 перекрашивает hover>.
```

## Отчёт: bulk

```text
Прогон по ds/components.md завершён.

Расширено:
- Button: Type × State × Size (3 × 4 × 3 = 36 ячеек, 2 прогона), +3 Semantic
- Input: State (5 ячеек), +1 Semantic

Пропущено:
- Toast — «достаточно текущей матрицы»
- Tooltip — пропущен пользователем

ds/components.md обновлён по всем расширенным.
ds/foundation.md: добавлено <K> Semantic-токенов.
```
