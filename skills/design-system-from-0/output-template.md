# Шаблоны выходных данных

`ds/` означает `workspace/product/ds/`.

| Файл | Фаза | Назначение |
|---|---|---|
| `ds/brand-brief.md` | 1 | Пять ответов брифа |
| `ds/foundation.md` | 2 | Два слоя токенов |
| `ds/components.md` | 3 | Каталог 7 базовых компонентов с Node ID |
| `ds/CONTRACT.md` | 4 | Правила работы с ДС |

Корневые `CLAUDE.md` и `AGENTS.md` не меняются: агент находит контракт по правилу в `AGENTS.md`.

---

## ds/brand-brief.md

```markdown
# Brand brief

**Продукт:** <product>
**Дата:** <ISO-дата>

| # | Вопрос | Ответ |
|---|---|---|
| 1 | Отрасль | <финтех / e-com / edtech / SaaS / медиа / госуслуги / другое> |
| 2 | Аудитория | <b2b / b2c / внутренний продукт> |
| 3 | Тон | <прилагательное, прилагательное> |
| 4 | Платформа | <web / mobile / web+mobile> |
| 5 | Локаль | <ru / en / ru+en> |

**Одной строкой:** <отрасль>, <аудитория>, <тон>, <платформа>, <локаль>
```

---

## ds/foundation.md

```markdown
# Foundation

**Бриф:** <одной строкой из brand-brief.md>

## Слой 1 — примитивные токены

### Палитра
- accent-500 = #2563EB  ← основной акцент
- accent-100 = …, accent-300 = …, accent-700 = …, accent-900 = …  ← оттенки акцента
- gray-00 = #FFFFFF, gray-50 = …, gray-100 = …, gray-200 = …, gray-300 = …, gray-500 = …, gray-700 = …, gray-900 = …, gray-950 = …  ← 9 нейтралов
- success-500 = …, warning-500 = …, error-500 = …, info-500 = …  ← функциональные

### Типографика
- font-sans = Inter
- size-xs = 12, size-sm = 14, size-base = 16, size-lg = 18, size-xl = 20, size-2xl = 24, size-3xl = 32, size-4xl = 48
- weight-regular = 400, weight-medium = 500, weight-semibold = 600, weight-bold = 700
- line-height-base = 1.5  ← +5% от 1.43 под кириллицу

### Радиусы
- radius-none = 0, radius-sm = 4, radius-md = 8, radius-lg = 16

### Отступы
- space-1 = 4, space-2 = 8, space-3 = 12, space-4 = 16, space-6 = 24, space-8 = 32, space-12 = 48, space-16 = 64

### Тени
- shadow-sm = 0 1 2 rgba(0,0,0,0.06)
- shadow-md = 0 4 12 rgba(0,0,0,0.08)
- shadow-lg = 0 12 32 rgba(0,0,0,0.12)

## Слой 2 — смысловые токены

### Поверхности
- surface-default → gray-00
- surface-subtle → gray-50
- surface-action-primary → accent-500
- surface-action-primary-hover → accent-700

### Текст
- text-default → gray-900
- text-muted → gray-500
- text-on-action → gray-00
- text-error → error-500

### Обводки
- border-default → gray-200
- border-strong → gray-300
- border-error → error-500

### Состояния
- bg-success → success-500
- bg-warning → warning-500
- bg-error → error-500
- bg-info → info-500

## Слой 3 — компонентные токены
*На стартовом наборе не нужен. Добавляется отдельной задачей при расширении компонентов вариантами и состояниями (button-padding-y, input-height и т.д.).*
```

---

## ds/components.md

```markdown
# Components

**Figma-файл:** <ссылка>
**Секция:** Foundation (<Node ID>)

| Компонент | Назначение | Варианты | Тип | Node ID |
|---|---|---|---|---|
| Button | Основное действие | State=primary / secondary / ghost | COMPONENT_SET | <id> |
| Input | Ввод текста | State=default / error | COMPONENT_SET | <id> |
| Card | Контейнер с заголовком и текстом | — | COMPONENT | <id> |
| Modal | Диалог с шапкой и кнопками | — | COMPONENT | <id> |
| Navbar | Горизонтальная навигация | — | COMPONENT | <id> |
| IconButton | Квадратная кнопка под иконку | — | COMPONENT | <id> |
| Badge | Статус | Variant=success / warning / error / info | COMPONENT_SET | <id> |

## TextStyles
DS/Heading/4xl, DS/Heading/3xl, DS/Heading/2xl, DS/Heading/xl, DS/Heading/lg, DS/Body/base, DS/Body/sm, DS/Body/sm Medium, DS/Body/xs, DS/Label/xs
```

---

## ds/CONTRACT.md

```markdown
# DS Contract — правила работы с дизайн-системой продукта <product>

**Источник ДС:** ds_baseline (стартовая, поставлена с нуля)
**Figma-файл:** <ссылка на тот же файл, где wireframes>
**Дата постановки:** <ISO-дата>
**Артефакты:** workspace/product/ds/

## Главное правило

При любом задании, которое касается интерфейса продукта (новый экран, форма, карточка, доработка существующего), агент работает строго через эту ДС. Новые экраны = композиция из существующих компонентов и существующих токенов. Прибитые цвета, размеры, радиусы и шрифты в обход Variables запрещены.

## Что значит «через ДС»

1. **Компоненты — только из UI-кита.** При сборке экрана агент берёт инстансы из секции «Foundation» (на старте: Button, Input, Card, Modal, Navbar, IconButton, Badge) и не рисует новые с нуля под конкретный экран.
2. **Цвета — только через Semantic Variables.** Никаких `fills = SOLID #2563EB`, только привязка к Variable (например, `surface-action-primary`). Прибитый hex считается багом и переделывается через Variable.
3. **Типографика — через TextStyles `DS/…`.** Размеры, межстрочный и веса берутся из Primitive, семейство шрифта — `font-sans`.
4. **Радиусы и отступы — через Variables.** `cornerRadius`, `itemSpacing`, `padding` задаются через `setBoundVariable`, не прибитыми числами.
5. **Auto Layout везде.** Любая секция экрана собирается в Auto Layout, иначе при правке контента вёрстка поедет.

## Что делать, если нужного нет

**Нет компонента под задачу** (например, понадобился Tooltip):
1. Добавить компонент в секцию «Foundation» по тем же правилам: имя латиницей, fills через Semantic, Auto Layout.
2. Записать строку в `components.md`: имя, назначение, варианты, Node ID.
3. Только после этого использовать его на экране.

**Нет смыслового токена** (например, нужен `surface-warning-subtle`):
1. Добавить Variable в Semantic с привязкой к существующему примитиву.
2. Если подходящего примитива нет, добавить его в Primitive.
3. Записать в `foundation.md` в соответствующий слой.
4. Только после этого использовать на экране.

**Нет паттерна композиции** (например, типовая «карточка с картинкой, телом и кнопками»):
1. Завести `patterns.md` рядом с этим файлом, если его нет.
2. Описать паттерн: имя, из каких компонентов и токенов собирается, при желании Node ID примера.
3. Использовать как референс при сборке экранов.

## Синхронизация Figma ↔ md

`foundation.md` и `components.md` — индекс, по которому работает агент. Расхождение индекса с Figma ломает следующие сборки.

- Правишь Variables или компоненты руками в Figma → сразу обновляешь соответствующий md-файл.
- Правишь md-файл и хочешь отразить это в Figma → перезапускаешь соответствующую фазу (`skills/design-system-from-0` для фундамента и базовых компонентов, отдельная задача для расширения UI-кита).
- Не редактируешь Variables руками без обновления `foundation.md`, иначе следующий прогон откатит правки.

## Что запрещено

- Прибитые цвета (`fill = SOLID hex`) в новых компонентах или экранах.
- Прибитые размеры шрифта, line-height, радиусы и отступы там, где есть подходящий Variable.
- Создание нового компонента «на лету» в составе экрана, если его можно вынести в UI-кит.
- Перезапись существующих Variables новыми значениями без согласования (это ребрендинг, отдельный процесс).
- Вторая параллельная ДС для этого же продукта.

## Когда правила можно нарушить

Только по явной команде пользователя «здесь без ДС, ad-hoc» (например, одноразовый рендер для презентации). Тогда делается отдельный фрейм вне секции «Foundation» с пометкой «ad-hoc, не часть продукта».
```

---

## Промежуточный отчёт после Figma

```text
Стартовая ДС в Figma поставлена:
- Variables: Primitive (N токенов) + Semantic (M токенов с привязками)
- Секция «Foundation»: палитра, типографика, радиусы, отступы
- 7 базовых компонентов: Button, Input, Card, Modal, Navbar, IconButton, Badge
- 10 TextStyles DS/…, все text-узлы привязаны
- Все fills и обводки привязаны к Semantic

Перехожу к контракту ДС.
```

---

## Финальный отчёт

```text
ДС поставлена и закреплена контрактом.

Артефакты (workspace/product/ds/):
  brand-brief.md   — бриф
  foundation.md    — токены (два слоя)
  components.md    — каталог 7 базовых компонентов
  CONTRACT.md      — правила работы с ДС (агент читает его автоматически)

В Figma:
  Две коллекции Variables (Primitive, Semantic) с привязками
  Секция «Foundation» рядом с «Wireframes — low-fi»
  7 базовых компонентов с привязками к Semantic

Дальше любая команда «собери экран», «добавь форму» уходит на компоненты UI-кита и Variables. Если чего-то не хватает, это сначала добавляется в ДС, а потом используется на экране.

Расширение UI-кита (Table, Tabs, Tooltip, Dropdown), варианты и состояния (hover, disabled, loading) — отдельные задачи поверх этой.

Не подтверждено / открытые вопросы: <если есть>
```
