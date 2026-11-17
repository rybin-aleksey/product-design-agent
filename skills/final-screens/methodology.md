# Методология: сборка финальных экранов

Фазы идут строго по порядку: следующая начинается только после гейта предыдущей (см. [checklist.md](checklist.md)).
`ds/` означает `workspace/product/ds/`, `task/` — `workspace/product/tasks/<TASK-ID>/`.

**Про инструмент:** `use_figma` — для чтения и записи; `get_screenshot` — для показа. Перед первым вызовом — `create_session` с `fileKey` и загрузка скилла `figma-use`; для сборки экранов также полезен скилл `figma-generate-design`.

---

## Принципы

### 1. Sizing — это смысл, не размер

У Auto Layout три режима по каждой оси: FIXED, HUG, FILL. `FIXED × FIXED` по умолчанию для секций даёт жёсткий каркас, который ломается при изменении контента.

| Уровень | Width | Height |
|---|---|---|
| Screen (корневой, 1440) | FIXED | HUG (FIXED только для фиксированного канваса) |
| Top-section: Header / Body / PageHeader / Toolbar / Banner | FILL | HUG (Header может быть FIXED) |
| Колонка в Body | sidebar / summary — FIXED, основная — FILL (grow = 1) | FILL или HUG |
| Row (StatsRow, ActionRow, Filters) | FILL | HUG |
| Spacer | FILL (grow = 1) | FILL или HUG |
| Card в горизонтальной сетке | FIXED по сетке или FILL для равных | HUG |
| Button / Badge / Chip / Input | HUG | HUG (или FIXED по высоте) |
| Modal | FIXED по дизайну | HUG |
| Иконка | FIXED | FIXED |

API: `node.layoutSizingHorizontal / layoutSizingVertical = "FILL" | "HUG" | "FIXED"` — ставится **после** `appendChild` в родителя с Auto Layout. `primaryAxisSizingMode = "FIXED"` + `resize(w, h)` для секций не использовать.

### 2. Компоненты — instance

Компонент берётся по Node ID из `components.md`. Для ComponentSet — инстанс дефолтного варианта и переключение свойств:

```js
const set = await figma.getNodeByIdAsync("1:234");
const inst = set.type === "COMPONENT_SET" ? set.defaultVariant.createInstance() : set.createInstance();
inst.setProperties({ Type: "ghost", Size: "md" }); // имена осей — как в components.md
```

Никаких `clone()` мастера и ручных копий геометрии: только instance сохраняет связь с ДС, и правки в ДС переезжают на экраны.

### 3. Variables и TextStyles

`fills`, `strokes` — `setBoundVariableForPaint` к Semantic; `padding*`, `itemSpacing`, радиусы — `setBoundVariable` к `space-*` / `radius-*`; текст на уровне экрана — `setTextStyleIdAsync` к TextStyles ДС, цвет текста — к `text-*`. На инстансах всё наследуется из мастера — проверяй, что override не вернул прибитый hex.

### 4. Иконки — реальные SVG

Библиотека фиксируется в фазе 0. SVG берётся из выбранной библиотеки (например, пакет `lucide-static` на unpkg) и рендерится сразу:

```js
const icon = figma.createNodeFromSvg(svgString);
icon.name = "Icon/chevron-down";
icon.resize(16, 16);
for (const v of icon.findAll(n => n.type === "VECTOR")) {
  if (v.strokes.length) v.strokes = [paint("text-muted")];
  if (v.fills.length) v.fills = [paint("text-muted")];
}
```

Если одна иконка встречается на экранах часто — это кандидат в компонент ДС (страница / секция Icons), по принципу 8.
Плейсхолдер-прямоугольник (fill `border-default`, имя `Placeholder/<что>`) допустим только для логотипа бренда, реального изображения, QR-кода. В composition map иконка называется конкретно: `Lucide / chevron-down 16`.

### 5. Тени — база

Тени в ДС — EffectStyles (`shadow-sm / md / lg`). Применяются через `await node.setEffectStyleIdAsync(style.id)`. Маппинг по умолчанию:

| Элемент | Тень |
|---|---|
| Card (interactive / hover) | shadow-sm или shadow-md |
| Modal | shadow-lg |
| Toast | shadow-sm (обычно уже в мастере) |
| Dropdown / Select panel | shadow-md |
| Sticky sidebar / summary | shadow-sm |

Если у мастера тень уже есть — на инстансе не дублировать.

### 6. Информативность

Overview / dashboard с контентом только в верхних 30–40% — composition map неполный. Ориентир — 60% площади занято осмысленным контентом. Типовые секции дашборда: топ-метрика, мини-график (sparkline / progress / trend), список основных сущностей с богатой мета, quick actions, recent activity, рекомендации. Какие секции нужны продукту — решает пользователь; агент предлагает.

### 7. Структурированная мета

Антипаттерн: `Ubuntu 22.04 · Амстердам · 2 CPU / 4 GB · uptime 12 дней` одной строкой. Правильно — каждое поле своим элементом: иконка + текст, флаг + текст, chips с иконками, muted-блок. FAQ, help-links, «подробнее» — кликабельные: chevron-right, underline или accordion.

### 8. Повторяющиеся блоки — компонент

| Копий на экране | Решение |
|---|---|
| 3+ | Выносим в UI-кит в этом же прогоне (почти всегда) |
| 2 | Спорно — спрашиваем пользователя |
| 1 | Локальная композиция |

Локальные копии повторяющегося блока — антипаттерн: правку придётся вносить в каждую копию, и часть забудется.

---

## Выбор режима

- Есть имя экрана → **single**: фазы 0–6.
- Имени нет → **bulk**:
  1. Read-only: фреймы секции «Wireframes» и «Screens».
  2. Уже собранные (`Screen/<Name>` есть в «Screens») — пропуск; пересборка только по явному запросу.
  3. По каждому несобранному — короткий бриф: «Wireframe Dashboard: 4 секции — Header / Sidebar / Stats Row / Files Table. Собираем? Какие edge cases закладываем?»
  4. «Да» → фазы 1–5 для экрана (фаза 0 — один раз на весь bulk). «Нет» / «пропустить» → следующий, причина фиксируется для отчёта.
  5. После последнего — bulk-отчёт.

---

## Фаза 0. Ресурсы

### 0.1. Библиотека иконок

Если нет `ds/icons.md` и строки «Icon library» в `ds/CONTRACT.md` — спроси:

> Какая библиотека иконок? Lucide / Phosphor / Heroicons / Material / собственный набор? Рекомендую Lucide: MIT, единый stroke-стиль, легко рендерится из SVG.

Предложи зафиксировать выбор строкой `Icon library: <выбор>` в `ds/CONTRACT.md` и допиши только после согласия. Для собственного набора — попроси ссылку на страницу / секцию иконок в Figma: тогда иконки ставятся инстансами.

### 0.2. Тени

Проверь, есть ли в `ds/foundation.md` тени и в файле — EffectStyles `shadow-*` (read-only `getLocalEffectStylesAsync()` в фазе 1.1). Если нет — сообщи пользователю и предложи добавить в Foundation по контракту. Тихо не пропускать.

---

## Фаза 1. Read

### 1.1. Wireframe в Figma

Один read-only вызов: найти фрейм по имени в секции «Wireframes», прочитать структуру на 2–3 уровня (имена, тип, Auto Layout, тексты-подписи), заодно EffectStyles и наличие секции «Screens».

```js
const page = figma.currentPage;
const wf = page.findOne(n => n.type === "SECTION" && n.name.startsWith("Wireframes"));
const frame = wf?.findOne(n => n.type === "FRAME" && n.name === "<ScreenName>");
if (!frame) return { error: "wireframe not found", available: wf?.children.map(c => c.name) };
const walk = (n, d) => ({
  name: n.name, type: n.type,
  layout: n.layoutMode, w: Math.round(n.width), h: Math.round(n.height),
  text: n.type === "TEXT" ? n.characters.slice(0, 80) : undefined,
  children: d < 3 && "children" in n ? n.children.map(c => walk(c, d + 1)) : undefined,
});
return {
  wireframe: { id: frame.id, tree: walk(frame, 0) },
  screens: page.findOne(n => n.type === "SECTION" && n.name === "Screens")?.children.map(c => c.name) ?? null,
  effects: (await figma.getLocalEffectStylesAsync()).map(s => s.name),
};
```

Фрейма нет — спроси: «Wireframe `<ScreenName>` не найден (есть: …). Это другое имя, или wireframe ещё не сделан?» Без wireframe не собираем, если пользователь прямо не сказал «собирай по описанию».

### 1.2. Каталог

Прочитай `ds/components.md` целиком: имя, Node ID, оси вариантов, назначение, `used in`.

### 1.3. Сверка покрытия

Для каждой секции wireframe подбери компонент из каталога:

```text
Секция wireframe     → Кандидат из ДС
Header (logo + nav)  → Navbar + Placeholder/Logo
Stats card           → Card + Badge
Files table          → Table + Pagination
```

Нет кандидата — спроси (шаблон в [output-template.md](output-template.md)): вынести в UI-кит (рекомендуется; для одиночного блока — отдельный прогон design-system-grow), взять ближайший, оставить плейсхолдер. Без решения дальше не идём.

### 1.4. Анализ достаточности wireframe

Обязательная фаза: wireframe низкой детализации, перенос 1:1 даёт сырой экран. Проверь каждую секцию:

1. **Семантическая ёмкость** — сущность раскрыта (статус, ключевые атрибуты, действия)? (Принцип 7)
2. **Пустые зоны** — overview / dashboard заполнен на ~60%? (Принцип 6)
3. **Edge cases** — empty / loading / error / overflow заложены?
4. **Уровень действий** — есть ли быстрые действия, кроме главного потока?
5. **Голый текст** — FAQ / help / breadcrumbs / footer-links кликабельны?
6. **Списки с одним элементом** — для реалистичности 3+ элемента с разными статусами и длиной данных.

Выдай сводку пунктами (шаблон в [output-template.md](output-template.md)) и получи решение по каждому. «Нет» / «оставь как есть» — не настаивай. Предложения опирай на контекст продукта; если данных о продукте не хватает — так и скажи и предложи вариант как гипотезу.

### 1.5. Повторяющиеся блоки

По предполагаемой композиции найди блоки 2+ копий (карточки списка, stat-карточки, шаги, action-chips, строки активности, уведомления). Сначала проверь, нет ли подходящего компонента в каталоге. По каждому — вопрос: выносим / нет / отложить, с предлагаемыми вариантами и где ещё может пригодиться.

- «Выносим» → очередь фазы 2.5 (для корпоративной ДС — локальный паттерн в `ds/patterns.md`).
- «Оставить локально» → локальная композиция, в composition map пометка «кандидат на вынос при появлении на втором экране».
- Если блоку нужны новые Semantic — отметь, они уйдут в фазу 2.5.

---

## Фаза 2. Composition map

Создай `task/artifacts/design/screens/<screen-name>.md` (kebab-case) по шаблону из [output-template.md](output-template.md). Карта описывает не только компоненты, но и визуальную детализацию данных:

- секции сверху вниз: Auto Layout (направление, gap, padding через `space-*`), sizing по таблице, фон, тень;
- содержимое: компонент + Node ID + значения вариантов, иконки по имени, демо-тексты;
- для каждой карточки списка — поля и их визуальная форма (icon-text / chip / badge / muted-meta);
- edge cases: где лежат, что показывают, `visible: false`;
- использованные токены, TextStyles, EffectStyles, иконки, плейсхолдеры;
- решения фаз 1.3–1.5.

Покажи файл целиком и дождись апрува. Правки → переписать → показать снова.
Если пользователь уходит в детали ниже уровня экрана (цвет, размер шрифта в Stats) — напомни: это уровень компонентов ДС, на экране подтянется автоматически; точечно — override после прогона или design-system-variants.

---

## Фаза 2.5. Вынос в UI-кит

Только если в фазе 1.5 что-то решено выносить. Происходит до сборки экрана, чтобы в фазе 4 ставить инстансы новых мастеров. Правила — как в [design-system-grow](../design-system-grow/methodology.md) (фазы 2–4) в сокращённом цикле.

1. **Брифы** компонентов в `task/analysis/final-screens-extract-<name>.md` по шаблону grow (назначение, анатомия, 2–3 варианта, привязки). Показать пакетом, апрув.
2. **Новые Semantic** — списком, только alias к существующему Primitive (`text-trend-positive → success-500`). Апрув.
3. **Figma:** токены — отдельным мутирующим вызовом (`createVariable` + `VARIABLE_ALIAS` во всех режимах коллекции); компоненты — пакетами ≤ 3 в секцию «UI Kit — extended» (создать рядом с «Foundation», если её нет); после каждого пакета — read-only проверка: тип `COMPONENT_SET` / `COMPONENT`, нет SOLID без привязки, у всех текстов `textStyleId`.
4. **md:** записи в `ds/components.md` (с `used in`), токены в `ds/foundation.md`, в composition map — Node ID новых компонентов вместо «локальной композиции».

---

## Фаза 3. Pre-canvas check

1. Каждый компонент из карты есть в `ds/components.md` (или создан в 2.5). Призрак — стоп, назад в 1.3 / 1.5.
2. Фаза 1.4 закрыта: по всем пунктам есть решение.
3. Фаза 1.5 закрыта; если был вынос — 2.5 завершена, md обновлены.
4. У каждой секции заданы gap / padding. Нет — предложи значение (`space-4`) вопросом.
5. Sizing по таблице, нет `FIXED × FIXED` у секций.
6. Иконки названы конкретно, ни одной «icon placeholder». Иначе — назад в 0.1.
7. Тени расписаны для Card / Modal / Toast / Dropdown / sticky-блоков.
8. Информативность: overview заполнен, мета структурирована, ссылки кликабельны (или пользователь явно отказался).
9. Edge cases прописаны или пользователь явно отказался («в продакшне это придётся доделывать»).
10. `fileKey` известен, сессия создана.

---

## Фаза 4. Canvas

### 4.1. Секция «Screens»

Нет — создать справа от «Wireframes» с зазором 80. Есть — использовать.

### 4.2. Сборка

Один мутирующий вызов на экран. Если экран большой и вызов не помещается — первый вызов создаёт `Screen/<Name>` со скелетом секций, следующие наполняют секции по одной-две (каждый находит фрейм по id). Без `throw` в конце мутирующих вызовов.

```js
const page = figma.currentPage;
const screens = page.findOne(n => n.type === "SECTION" && n.name === "Screens");
const vars = await figma.variables.getLocalVariablesAsync();
const v = n => vars.find(x => x.name === n);
const paint = n => figma.variables.setBoundVariableForPaint({ type: "SOLID", color: { r: 0, g: 0, b: 0 } }, "color", v(n));
const textStyles = await figma.getLocalTextStylesAsync();
const ts = n => textStyles.find(s => s.name.replace(/\s*\/\s*/g, "/") === n);
const effects = await figma.getLocalEffectStylesAsync();
const fx = n => effects.find(s => s.name === n);

const instanceOf = async (id, props) => {
  const n = await figma.getNodeByIdAsync(id);
  const inst = n.type === "COMPONENT_SET" ? n.defaultVariant.createInstance() : n.createInstance();
  if (props) inst.setProperties(props);
  return inst;
};
const frame = (name, dir, gap, pad) => {
  const f = figma.createFrame();
  f.name = name; f.layoutMode = dir; f.fills = [];
  if (gap) f.setBoundVariable("itemSpacing", v(gap));
  if (pad) for (const k of ["paddingTop", "paddingBottom", "paddingLeft", "paddingRight"]) f.setBoundVariable(k, v(pad));
  return f;
};
const add = (parent, child, w = "FILL", h = "HUG") => {
  parent.appendChild(child);
  child.layoutSizingHorizontal = w; child.layoutSizingVertical = h;
  return child;
};

const screen = frame("Screen/Dashboard", "VERTICAL");
screen.fills = [paint("surface-default")];
screens.appendChild(screen);
screen.resize(1440, 900);
screen.layoutSizingHorizontal = "FIXED"; screen.layoutSizingVertical = "HUG";

const header = add(screen, frame("Header", "HORIZONTAL", "space-6", "space-4"));
header.primaryAxisAlignItems = "SPACE_BETWEEN";
add(header, await instanceOf("<Navbar id>"), "FILL", "HUG");

const body = add(screen, frame("Body", "HORIZONTAL", "space-6", "space-6"));
const main = add(body, frame("Main", "VERTICAL", "space-6"), "FILL", "HUG");
const stats = add(main, frame("Stats Row", "HORIZONTAL", "space-4"));
for (let i = 0; i < 3; i++) {
  const card = add(stats, await instanceOf("<Card id>"), "FILL", "HUG");
  await card.setEffectStyleIdAsync(fx("shadow-sm").id);
}

// Edge case — сосед в той же секции, скрыт
const empty = add(main, frame("Empty state", "VERTICAL", "space-4", "space-12"));
empty.visible = false;
// ...
```

Правила:

1. Компоненты ДС — только `instanceOf` по Node ID, варианты через `setProperties`. Текстовые и boolean-свойства инстансов — тоже через `setProperties`, не через детач.
2. Auto Layout на экране и каждой секции; sizing — по таблице, после `appendChild`.
3. Фоны и обводки экрана и секций — Semantic; padding / gap / радиусы — `space-*` / `radius-*`; тексты вне инстансов — TextStyles ДС + цвет `text-*`.
4. Тени — EffectStyles по маппингу, без дублей с мастером.
5. Иконки — SVG выбранной библиотеки с цветом через Semantic (или инстансы собственного набора).
6. Плейсхолдеры — только для логотипа / изображения / QR: прямоугольник `Placeholder/<что>`, fill `border-default`.
7. Главный фрейм — `Screen/<Name>`; секции — `Header`, `Sidebar`, `Main`, `Stats Row`, `Files Table`.
8. Edge cases — соседи в той же секции, `visible = false`.
9. Демо-тексты — из composition map, на языке локали продукта.

### 4.3. Проверка после записи

Отдельный read-only вызов по `Screen/<Name>`:

1. Все узлы, соответствующие компонентам из карты, — `INSTANCE` с `mainComponent` из ожидаемого набора; нет `FRAME` на их месте.
2. У экрана и секций fill привязан к Variable; нет SOLID без `boundVariables.color` вне инстансов; у инстансов нет override с прибитым цветом.
3. У всех фреймов-секций `layoutMode !== "NONE"`; нет секций `FIXED × FIXED`, кроме разрешённых таблицей.
4. Тексты вне инстансов — с `textStyleId`.
5. Edge cases на месте и скрыты.
6. Плейсхолдеров не больше, чем в карте.

Затем `get_screenshot` по экрану — показать пользователю. Проблемный блок пересобрать, проверку повторить. Записать Final Screen Node ID в composition map.

---

## Фаза 5. Запись в каталог

### 5.1. `ds/components.md`

Каждому компоненту, использованному на экране, — строка `**Used in:**` с `<TASK-ID>/screens/<name>` (дописать через запятую, если строка есть). Остальное в записях не трогать.

### 5.2. Индекс экранов

`task/artifacts/design/screens/_index.md` — строка на экран: файл, назначение, Final Screen Node ID. Создать, если нет; дописывать, если есть.

---

## Фаза 6. Отчёт

Single или bulk — по шаблонам из [output-template.md](output-template.md). Отдельно: что проверено по факту, какие плейсхолдеры остались на замену руками, какой демо-контент требует реальных данных.

---

## Синхронизация Figma ↔ md

- Раскладку правили руками в Figma — обнови composition map, иначе следующий прогон вернёт старую структуру.
- Компоненты меняются в ДС, а не на экране: экраны подтянут правки через инстансы.
- Черновики `analysis/final-screens-extract-*.md` можно удалить после прогона только с согласия пользователя.
