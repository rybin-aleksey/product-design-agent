# Методология: стартовая ДС с нуля

Четыре фазы. Порядок строгий: каждая следующая начинается только после гейта предыдущей (см. [checklist.md](checklist.md)).
`ds/` означает `workspace/product/ds/`. Все файлы ДС создаются только там.

---

## Фаза 1. Бриф

Задай пять вопросов одним блоком. Без уточнений и философии, ответы короткие.

```text
1. Отрасль продукта. Одно слово: финтех / e-com / edtech / SaaS / медиа / госуслуги / другое.
2. Аудитория. b2b / b2c / внутренний продукт.
3. Тон. Два прилагательных через запятую: например «строгий, надёжный» или «дружелюбный, простой».
4. Платформа. web / mobile / web+mobile.
5. Локаль. ru / en / ru+en.
```

- Если ответ уже есть в `workspace/product/context/` (например, аудитория в `target-audience.md`), подставь его и попроси подтвердить, а не спрашивай заново.
- Если пользователь отвечает полстраницы, выдели ключевое и переспроси: «зафиксировал X, верно?» Не превращай бриф в интервью.
- Сохрани ответы в `ds/brand-brief.md` (шаблон в [output-template.md](output-template.md)). Это якорь решений по тону палитры, типографике и плотности отступов.

---

## Фаза 2. Foundation в Markdown

Собери `ds/foundation.md` по шаблону из [output-template.md](output-template.md). Значения привязывай к брифу: тон определяет акцент, платформа задаёт шкалу, аудитория задаёт плотность.

### Зашитый минимум

Состав фиксирован, значения адаптируются под бриф.

| Группа | Состав |
|---|---|
| Палитра | 1 акцент (оттенки 100, 300, 500, 700, 900), 9 нейтралов (00, 50, 100, 200, 300, 500, 700, 900, 950), 4 функциональных (success, warning, error, info) |
| Типографика | Inter (или альтернатива с кириллицей при `ru`), шкала xs 12 … 4xl 48, веса 400/500/600/700, межстрочный +5% при кириллице |
| Радиусы | 0, 4, 8, 16 |
| Отступы | 4, 8, 12, 16, 24, 32, 48, 64 |
| Тени | 3 уровня (sm, md, lg): мягкие, без бренд-цвета |

### Кириллица (локаль ru или ru+en)

- Шрифт с поддержкой Щ/Ъ/Ь/Ы: Inter, IBM Plex Sans, Roboto, PT Root UI, Onest, Manrope, Ubuntu.
- Межстрочный на 3–5% больше базового.
- На крупных заголовках межбуквенный от −0.5 до −1%.
- Для `ru+en` проверь, что основной и резервный шрифты покрывают оба алфавита.

### Три слоя токенов

1. **Примитивные:** сырые значения (`accent-500 = #2563EB`).
2. **Смысловые:** роль в интерфейсе, всегда ссылка на примитив (`surface-action-primary → accent-500`).
3. **Компонентные:** на стартовом наборе не нужны. Добавляются отдельной задачей при появлении вариантов и состояний (`button-padding-y`, `input-height`).

### Гейт

Покажи файл пользователю и дождись апрува. Если бриф меняет что-то существенное (например, `ru+en` требует резервный шрифт), внеси правки и переспроси.
**До апрува никаких MCP-вызовов.**

---

## Фаза 3. Перенос в Figma

### 3.0. Получение файла

Попроси: «Пришли ссылку на Figma Design-файл, в котором собирается продукт (если есть wireframes, то тот же файл). Variables и фундамент ставим туда же».

- Извлеки `fileKey` (`figma.com/design/:fileKey/...`; для веток бери `branchKey`).
- Вызови `create_session` с `fileKey`, загрузи скилл `figma-use` и только после этого вызывай `use_figma`.

### 3.1. Первый вызов `use_figma`: Variables

Один вызов. Создаёт две коллекции и заливает значения из `foundation.md`.

```js
// Коллекция 1: Primitive
const primitive = figma.variables.createVariableCollection("Primitive");
const P = {};

// Цвета
P["accent-500"] = figma.variables.createVariable("accent-500", primitive, "COLOR");
P["accent-500"].setValueForMode(primitive.defaultModeId, {r: 0.145, g: 0.388, b: 0.922});
// ... остальные цвета слоя 1

// Числа: радиусы, отступы, размеры шрифта, веса
P["radius-md"] = figma.variables.createVariable("radius-md", primitive, "FLOAT");
P["radius-md"].setValueForMode(primitive.defaultModeId, 8);
// ... остальные числа

// Строки: семейство шрифта
P["font-sans"] = figma.variables.createVariable("font-sans", primitive, "STRING");
P["font-sans"].setValueForMode(primitive.defaultModeId, "Inter");

// Коллекция 2: Semantic
const semantic = figma.variables.createVariableCollection("Semantic");
const S = {};

S["surface-action-primary"] = figma.variables.createVariable("surface-action-primary", semantic, "COLOR");
S["surface-action-primary"].setValueForMode(
  semantic.defaultModeId,
  {type: "VARIABLE_ALIAS", id: P["accent-500"].id}  // привязка к примитиву
);
// ... остальные смысловые, все через VARIABLE_ALIAS
```

Правила:
- Порядок: сначала примитивы, потом смыслы. Смыслы ссылаются на примитивы только через `VARIABLE_ALIAS`, иначе ребрендинг сломается.
- Имена точно как в `foundation.md`.
- Hex переводится в RGB 0–1 (`#2563EB` → `{r: 0.145, g: 0.388, b: 0.922}`).
- На канвасе ничего не появляется: Variables живут в панели Local Variables.

Затем отдельным read-only вызовом проверь количество переменных в каждой коллекции и то, что все значения Semantic имеют тип `VARIABLE_ALIAS`. Попроси пользователя открыть Local Variables: при наведении на значение в Semantic должно показываться имя примитива.

### 3.2. Второй вызов `use_figma`: фундамент и базовые компоненты

Создаёт `SectionNode` «Foundation» рядом с «Wireframes — low-fi» (или на свободном месте страницы, если wireframes нет).
Если вызов получается слишком большим, раздели его на несколько мутирующих (стили и Foundation-блоки, затем компоненты). Правило отсутствия `throw` действует для каждого.

```text
Foundation (SectionNode, fill = surface-subtle)
├── Palette (Frame, Auto Layout)
│   ├── PrimitiveColors: 9 нейтралов + 5 оттенков акцента + 4 функциональных
│   └── SemanticColors: плитки смысловых токенов с подписью «имя → примитив»
├── Typography (Frame): строки шкалы с подписями «xs 12», «sm 14» … «4xl 48»
├── Radii (Frame): 4 квадрата с разными радиусами
├── Spacing (Frame): линейка 8 значений отступов
└── Components (Frame, Auto Layout, layoutWrap = WRAP, ниже блоков)
    ├── Button: State=primary / secondary / ghost
    ├── Input: State=default / error
    ├── Card: заголовок + текст
    ├── Modal: шапка + тело + кнопки
    ├── Navbar: горизонтальная навигация
    ├── IconButton: квадратная кнопка под иконку
    └── Badge: Variant=success / warning / error / info
```

#### Правила отрисовки

1. **Компоненты — настоящие `ComponentNode`.** Создаются через `figma.createComponent()` или `figma.createComponentFromNode(frame)`, видны в Assets и поддерживают `createInstance()`. Простые `createFrame()` для UI-элементов запрещены: на экранах их нельзя переиспользовать как инстансы.
   Варианты объединяются через `combineAsVariants`, имена задаются в формате `Property=value` до объединения:
   ```js
   const primary   = figma.createComponentFromNode(primaryFrame);   primary.name   = "State=primary";
   const secondary = figma.createComponentFromNode(secondaryFrame); secondary.name = "State=secondary";
   const ghost     = figma.createComponentFromNode(ghostFrame);     ghost.name     = "State=ghost";
   const buttonSet = figma.combineAsVariants([primary, secondary, ghost], page);
   buttonSet.name = "Button";
   componentsRow.appendChild(buttonSet);
   ```
   Компоненты без вариантов (Card, Modal, Navbar, IconButton) создаются одиночными `ComponentNode`.

2. **Типографика — через TextStyles.** Каждой ступени шкалы соответствует свой `TextStyle`. `fontSize` привязан к Primitive Variable, а `lineHeight` и `letterSpacing` задаются в `PERCENT`, чтобы не ломаться при смене размера:
   ```js
   await figma.loadFontAsync({family: "Inter", style: "Medium"});
   const style = figma.createTextStyle();
   style.name = "DS/Heading/lg";
   style.fontName = {family: "Inter", style: "Medium"};
   style.setBoundVariable("fontSize", P["size-lg"]);
   style.lineHeight = {value: 150, unit: "PERCENT"};
   await text.setTextStyleIdAsync(style.id);
   ```
   Минимальный набор стилей: `DS/Heading/4xl`, `DS/Heading/3xl`, `DS/Heading/2xl`, `DS/Heading/xl`, `DS/Heading/lg`, `DS/Body/base`, `DS/Body/sm`, `DS/Body/sm Medium`, `DS/Body/xs`, `DS/Label/xs` (letter-spacing 5%).
   Каждый text-узел в Foundation и в компонентах привязан к одному из этих стилей. Прибивать `fontSize` напрямую запрещено.
   **Цвет текста задаётся отдельно:** TextStyle хранит только типографику, а цвет идёт через `text.fills` с биндингом к Semantic (`text-default`, `text-muted`, `text-on-action`, `text-error`).
   **Gotcha:** Figma может нормализовать пробелы вокруг `/` в именах стилей. При поиске сравнивай `name.replace(/\s*\/\s*/g, "/")`.

3. **Все заливки и обводки — через Variables:**
   ```js
   const paint = figma.variables.setBoundVariableForPaint(
     {type: "SOLID", color: {r: 0, g: 0, b: 0}}, "color", S["surface-action-primary"]
   );
   node.fills = [paint];
   ```
   Прибитых hex в компонентах быть не должно. Если поле не поддерживает Variable (например, тени), используй EffectStyle.

4. **Auto Layout везде:** любая секция, любой компонент.

5. **Радиусы через Variables, каждый угол отдельно:**
   ```js
   for (const k of ["topLeftRadius", "topRightRadius", "bottomLeftRadius", "bottomRightRadius"]) {
     node.setBoundVariable(k, P["radius-md"]);
   }
   ```
   Отступы (`itemSpacing`, `padding*`) тоже биндятся к `space-*`.

6. **Имена латиницей по смыслу:** `Button`, `Input`, `Card`, `Modal`, `Navbar`, `IconButton`, `Badge`. Варианты называются `State=…` / `Variant=…`, тогда Properties-панель получает правильную схему.

7. **Раскладка:** Palette, Typography, Radii и Spacing идут сверху горизонтально, компоненты ниже в одном `Components`-фрейме с `layoutWrap = "WRAP"`. Между блоками `itemSpacing = 32` (биндинг к `space-8`).

8. **Фон SectionNode** привязан к `surface-subtle`, чтобы белые карточки компонентов читались.

9. **Проверка по факту, не по интенту** (см. 3.3).

10. **Никакого `throw` в конце мутирующего вызова.** `use_figma` исполняет вызов как транзакцию: `throw` откатывает все мутации. Отчёт делай отдельным read-only вызовом, а статус можно положить в `node.name` или `setSharedPluginData`. Симптом нарушения: лог говорит «applied=N», а в файле ничего не изменилось.

### 3.3. Проверка после записи

После каждого мутирующего вызова сделай **отдельный** read-only вызов:

1. `findOne` на `COMPONENT_SET` `Button`, `Input`, `Badge` и `COMPONENT` `Card`, `Modal`, `Navbar`, `IconButton`: правильные `type` и `name`. Если найден `null` или `FRAME`, это ошибка, исправь её до продолжения.
2. `getLocalTextStylesAsync()` возвращает 10 стилей с префиксом `DS/`. Если их меньше, `createTextStyle` откатился.
3. У всех text-узлов внутри Foundation непустой `textStyleId`. Если есть пустые, `setTextStyleIdAsync` откатился.
4. Ни у одного fill или stroke в Foundation нет SOLID без `boundVariables.color`.
5. `get_screenshot` по секции «Foundation» нужно показать пользователю.
6. Попроси пользователя проверить в Inspect, что fill у Button помечен как Variable, и что смена значения `accent-500` перекрашивает компоненты. Если что-то не перекрасилось, найди непривязанный fill и исправь его.

После проверки запиши `ds/components.md` с фактическими Node ID.

### 3.4. Краткий показ

Отчёт по шаблону «Промежуточный отчёт после Figma» из [output-template.md](output-template.md). Сразу переходи к фазе 4: без контракта ДС остаётся декорацией, и при сборке экранов агент не обязан её использовать.

---

## Фаза 4. Контракт ДС

Цель — закрепить правила работы с ДС на всю дальнейшую работу с продуктом. После этой фазы любая команда «собери экран X», «добавь форму», «дорисуй карточку» опирается на ДС, пока пользователь явно не скажет «забудь про ДС».

### 4.1. `ds/CONTRACT.md`

Создай по шаблону из [output-template.md](output-template.md), подставив ссылку на Figma-файл и дату.

Корневые `CLAUDE.md` и `AGENTS.md` не меняй: это файлы шаблона агента. `AGENTS.md` уже требует читать `workspace/product/ds/CONTRACT.md`, если он существует, поэтому любая будущая сессия найдёт контракт сама.

### 4.2. Финальный отчёт

По шаблону «Финальный отчёт» из [output-template.md](output-template.md).
