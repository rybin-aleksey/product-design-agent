# Методология: расширение матрицы вариантов и состояний

Фазы идут строго по порядку: следующая начинается только после гейта предыдущей (см. [checklist.md](checklist.md)).
`ds/` означает `workspace/product/ds/`.

**Про инструмент:** `use_figma` — для чтения и записи; `get_screenshot` — для показа результата. Перед первым вызовом — `create_session` с `fileKey` и загрузка скилла `figma-use`.

**Про имена вариантов.** Figma строит оси Component Set из имён ячеек вида `Property=value, Property=value` (`Type=primary, State=hover, Size=md`). Имена через слэш (`Button/Primary/Hover/Md`) осей не создают — получатся отдельные компоненты, сгруппированные в Assets. Поэтому в этом скилле ячейки именуются только через `Property=value`, как в `design-system-from-0` и `design-system-grow`.

---

## Выбор режима

- Есть имя компонента → **single**: фазы 1–6 для него.
- Имени нет → **bulk**:
  1. Прочитай `ds/components.md`.
  2. Отметь «неполные» компоненты: нет оси состояний (`state:`) или в ней только `default` / `default | error`; нет оси размеров (`size:`); всего вариантов ≤ 2.
  3. Покажи список одним блоком и иди по нему по очереди: «У Card сейчас 1 вариант, состояний нет. Расширяем? Если да — какие варианты / состояния / размеры?»
  4. «Да» → single-прогон фаз 1–5 для компонента. «Нет» / «пропустить» → следующий, причина фиксируется для отчёта.
  5. После последнего компонента — bulk-отчёт (фаза 6).

---

## Фаза 1. Read — что уже есть

### 1.1. Строка в `ds/components.md`

Найди запись компонента: имя, Node ID, текущая матрица (все оси вариантов).
Если записи нет — спроси: «Компонента `<Name>` нет в `ds/components.md`. Это новый компонент? Тогда нужен design-system-grow. Запустить?»

### 1.2. Компонент в Figma

Один read-only вызов `use_figma`:

```js
const node = await figma.getNodeByIdAsync("<Node ID>");
if (!node) return { error: "not found" };
const props = node.type === "COMPONENT_SET" || node.type === "COMPONENT"
  ? Object.fromEntries(Object.entries(node.componentPropertyDefinitions).map(([k, d]) => [
      k.replace(/#.*$/, ""), d.type === "VARIANT" ? d.variantOptions : d.type,
    ]))
  : null;
return {
  type: node.type,
  name: node.name,
  props,
  cells: node.type === "COMPONENT_SET" ? node.children.map(c => ({ id: c.id, name: c.name })) : [],
  // структура одной ячейки — шаблон для новых
  sample: (node.type === "COMPONENT_SET" ? node.defaultVariant : node).findAll(() => true)
    .slice(0, 30).map(n => ({ name: n.name, type: n.type, style: n.textStyleId || undefined })),
};
```

Сверь с `components.md`. При расхождении — покажи и спроси: «В Figma у Button 3 варианта, в components.md записано 2. Что считаем правдой?» Правдой по умолчанию не считается ни то, ни другое.

Зафиксируй `currentMatrix` — оси и значения, а также фактические имена осей (например, у Button из baseline ось типа называется `State`: `State=primary | secondary | ghost`). Если имя оси расходится со смыслом и в матрицу добавляется настоящая ось состояний — предложи переименовать ось (`State` → `Type`) через `editComponentProperty`: инстансы при этом сохраняются. Без согласия пользователя оси не переименовываются.

---

## Фаза 2. Brief — что добавляем

Покажи текущую матрицу одной строкой и задай три вопроса одним блоком (шаблон в [output-template.md](output-template.md)):

```text
Сейчас у <ComponentName>: <currentMatrix>

1. Варианты. Что добавить? (например: ghost; или пропускаем)
2. Состояния. Какие добавить? (например: hover, disabled, loading, focus; или пропускаем)
3. Размеры. Какие добавить? (например: sm, lg; или пропускаем)
```

- Любая категория может быть пустой. Все три пустые — «нечего добавлять», прогон закрыт.
- Неконкретный ответ («все нужные», «по умолчанию») — переспроси. Дефолтов нет: это продуктовое решение.
- Для мобильного продукта уточни, нужен ли `pressed` вместо `hover`.

---

## Фаза 3. md-апдейт

### 3.1. Новая матрица и её размер

`newMatrix = currentMatrix + добавления`. Размер — произведение числа значений по осям.

Пример: было `type: primary | secondary; state: default; size: md`, добавили `ghost`, `hover, disabled, loading`, `sm, lg` → `type: primary | secondary | ghost; state: default | hover | disabled | loading; size: sm | md | lg` = 3 × 4 × 3 = **36 ячеек**.

- **≤ 24 ячеек** — один прогон.
- **25+ ячеек** — предупреди и предложи дробление: «Матрица 36 ячеек — write-tools на пределе. Первый прогон: варианты × состояния (3 × 4 = 12), второй: добавляем размеры (до 36). Подтверждаешь?» При согласии фазы 3.3–4 повторяются дважды с промежуточным апрувом. Если пользователь настаивает на одном прогоне — соглашайся и предупреди, что за качество не ручаешься.

Не все ячейки обязаны существовать (например, `loading` только у `primary`). Если пользователь так решил — матрица неполная, перечисли явно, каких ячеек нет.

### 3.2. Semantic-токены под состояния

| Состояние | Что меняется | Какой Semantic нужен |
|---|---|---|
| `hover` / `pressed` | Фон action-элемента темнее на ~10–15% | `surface-action-<variant>-hover` / `-pressed` |
| `disabled` | Opacity 0.4–0.5 на том же surface | — |
| `loading` | Тот же surface, контент скрыт, плейсхолдер под спиннер | — |
| `focus` | Outline 2px по контуру | `border-focus-ring` |
| `selected` | Фон чуть отличается от default | `surface-<variant>-selected` |
| `error` (поля ввода) | Граница цвета ошибки | `border-error` (может уже быть) |

Для каждого нужного токена проверь `foundation.md`. Чего нет — предложи alias к Primitive (`surface-action-primary-hover → accent-700`, если `surface-action-primary → accent-500`). Нет подходящего Primitive — предложи сначала его (значение тоже на апрув).

Покажи список добавлений и дождись апрува. Без апрува дальше не идём.

### 3.3. Описание новой матрицы

Собери `workspace/product/tasks/<TASK-ID>/analysis/component-variants-<name>.md` по шаблону из [output-template.md](output-template.md):

1. Список ячеек: существующие (не трогаем) и новые, имена в формате `Property=value, …`.
2. Что меняется на каждом состоянии — fill / stroke / opacity / контент с именами токенов.
3. Что меняется на каждом размере — padding, gap, высота через `space-*`, TextStyle на размер (`DS/Body/sm` → `DS/Body/base`), размер плейсхолдера спиннера.

Покажи и дождись апрува.

---

## Фаза 4. Canvas — отрисовка

### 4.1. Semantic Variables в Figma

Если в 3.2 подтверждены новые токены — отдельный мутирующий вызов создаёт их в коллекции Semantic (и Primitive, если нужно) как `VARIABLE_ALIAS`:

```js
const cols = await figma.variables.getLocalVariableCollectionsAsync();
const vars = await figma.variables.getLocalVariablesAsync();
const semantic = cols.find(c => c.name === "Semantic");
const byName = n => vars.find(v => v.name === n);

const add = (name, aliasTo) => {
  if (byName(name)) return; // не дублируем
  const v = figma.variables.createVariable(name, semantic, "COLOR");
  v.setValueForMode(semantic.defaultModeId, { type: "VARIABLE_ALIAS", id: byName(aliasTo).id });
};
add("surface-action-primary-hover", "accent-700");
add("border-focus-ring", "accent-500");
```

Если у коллекции несколько режимов (light / dark) — задай alias в каждом режиме, значения для недефолтных режимов тоже на апрув.

### 4.2. Промоут одиночного компонента в Set

Если компонент — одиночный `ComponentNode`:
- переименуй его в первую ячейку новой схемы (`Type=default, State=default` и т.п.);
- `figma.combineAsVariants([component], component.parent)` — существующие инстансы остаются привязаны;
- задай Set'у прежнее имя компонента и перенеси `description`;
- запиши новый Node ID (Set) для `components.md`.

Делается в том же мутирующем вызове, что и 4.3, если матрица небольшая; иначе отдельным.

### 4.3. Достройка ячеек

Один мутирующий вызов на компонент (или на половину матрицы при дроблении).

```js
const set = await figma.getNodeByIdAsync("<setNodeId>");
const vars = await figma.variables.getLocalVariablesAsync();
const v = n => vars.find(x => x.name === n);
const styles = await figma.getLocalTextStylesAsync();
const style = n => styles.find(s => s.name.replace(/\s*\/\s*/g, "/") === n);
const paint = name => figma.variables.setBoundVariableForPaint(
  { type: "SOLID", color: { r: 0, g: 0, b: 0 } }, "color", v(name)
);

const existing = new Set(set.children.map(c => c.name));
const base = set.defaultVariant; // шаблон структуры

for (const cell of newCells) { // [{ name, type, state, size }, ...] из апрувнутого описания
  if (existing.has(cell.name)) continue;
  const comp = base.clone();     // наследует Auto Layout, слои, привязки
  comp.name = cell.name;         // "Type=primary, State=hover, Size=md"
  set.appendChild(comp);

  // Состояние
  if (cell.state === "hover") comp.fills = [paint(`surface-action-${cell.type}-hover`)];
  if (cell.state === "disabled") comp.opacity = 0.4;
  if (cell.state === "focus") {
    comp.strokes = [paint("border-focus-ring")];
    comp.strokeWeight = 2;
    comp.strokeAlign = "OUTSIDE";
  }
  // Размер: padding / gap через space-*, TextStyle по размеру
  // comp.setBoundVariable("paddingLeft", v("space-3")); ...
  // for (const t of comp.findAll(n => n.type === "TEXT")) {
  //   await figma.loadFontAsync(style("DS/Body/sm").fontName);
  //   await t.setTextStyleIdAsync(style("DS/Body/sm").id);
  // }
  // Loading: скрыть контент, добавить плейсхолдер под спиннер (12 / 16 / 20 по размеру)
}
// Раскладка ячеек Set'а в сетку — через Auto Layout самого Set'а (layoutWrap) или координаты
```

Правила отрисовки:

1. **Новые ячейки — `clone()` дефолтной ячейки**, а не `createComponent()` с нуля: так сохраняются структура, привязки и TextStyles. Существующие ячейки не пересоздаются.
2. **Fills и strokes — только `setBoundVariableForPaint` к Semantic.** `setBoundVariable("fills", …)` не работает для цвета. Тени — через существующие EffectStyle.
3. **Padding, gap, радиусы — `setBoundVariable` к `space-*` / `radius-*`.** Прибитые числа запрещены.
4. **Типографика по размеру — смена TextStyle**, не прибитый `fontSize`. Новые TextStyles не создаются; не хватает — сначала добавить в Foundation по контракту.
5. **Auto Layout в каждой ячейке.**
6. **Loading:** плейсхолдер (Ellipse 12 / 16 / 20 px по размеру, fill через Semantic) по центру, остальной контент скрыт через `visible = false`. Анимацию и interactive components не делаем — их ставят вручную.
7. **Focus:** stroke 2px, `strokeAlign = "OUTSIDE"`, цвет `border-focus-ring`.
8. **Disabled:** тот же surface + `opacity` 0.4–0.5, без отдельного токена.
9. **Никакого `throw` в конце мутирующего вызова** — откатывает всю транзакцию. Отчёт — отдельным read-only вызовом.

### 4.4. Проверка после записи

Отдельным read-only вызовом:

1. Узел — `COMPONENT_SET`, в `componentPropertyDefinitions` все оси и значения из `newMatrix`. Если оси не появились — имена ячеек не в формате `Property=value`.
2. Число ячеек = ожидаемому (с учётом явно исключённых).
3. Ни одного SOLID fill / stroke без `boundVariables.color` во всех ячейках.
4. Все text-узлы с непустым `textStyleId`.
5. Точечно: одна новая ячейка (например, `Type=primary, State=hover, Size=md`) — fill привязан к ожидаемому токену.
6. `get_screenshot` по Set'у — показать пользователю.

Ячейку с проблемой пересобрать через `use_figma`, затем повторить проверку.

---

## Фаза 5. Запись в каталог

### 5.1. `ds/components.md`

Замени строку «Варианты» у компонента на `newMatrix` (и Node ID, если был промоут). Под записью — короткая пометка о расширении: дата, что добавлено, новые Semantic (шаблон в [output-template.md](output-template.md)). Остальные записи не трогаем.

### 5.2. `ds/foundation.md`

Если токены добавлялись — допиши их в слой Semantic (и Primitive, если добавлялся) в формате `имя → примитив`. Существующие не дублируй и не правь.

---

## Фаза 6. Отчёт

Single или bulk — по шаблонам из [output-template.md](output-template.md). В отчёте отдельно: что проверено по факту, что осталось на ручную проверку (например, смена `accent-700` перекрашивает hover).

---

## Синхронизация Figma ↔ md

- Матрицу правили руками в Figma — обнови строку в `components.md`, иначе следующий прогон пропустит компонент как «уже расширенный» или посчитает ручные ячейки расхождением.
- Структурные правки — сначала в описании, затем повторная достройка через этот скилл.
- Черновики `analysis/component-variants-<name>.md` можно удалить после прогона, но только с согласия пользователя.
