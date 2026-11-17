# Методология: расширение UI-кита

Четыре фазы. Порядок строгий: каждая следующая начинается только после гейта предыдущей (см. [checklist.md](checklist.md)).
`ds/` означает `workspace/product/ds/`.

**Про инструмент:** `use_figma` — для записи на канвас и для проверочных чтений (`figma.variables.getLocalVariablesAsync()`, `getLocalTextStylesAsync()`); `get_variable_defs` — альтернатива для проверки привязок при аудите; `get_screenshot` — для показа результата.

**Про файл:** работаем в том же Figma Design-файле, где лежат wireframes и «Foundation». Секцию «UI Kit — extended» ставим рядом с «Foundation», не внутри: стартовые компоненты остаются ядром.
Если ДС подключена через ds_scan (своя или дублированная Community) и «Foundation» в файле нет — секция ставится на странице Components файла ДС, справа от существующих компонентов.

---

## Фаза 1. Каталог и выбор компонентов

### 1.1. Чтение каталога

Прочитай `ds/CONTRACT.md`, `ds/foundation.md`, `ds/components.md` (и `ds/source.md`, если есть). Выпиши:
- имена всех компонентов — чтобы не предложить дубль;
- какие смысловые разделы уже покрыты: Действия, Ввод, Контейнеры, Навигация, Фидбэк, Данные;
- источник ДС — для корпоративной прогон не запускается (см. SKILL.md);
- имена TextStyles и Semantic-токенов, доступных для новых компонентов.

Если `components.md` отсутствует или пуст — спроси: «`ds/components.md` пустой. Сначала нужен ds_baseline или ds_scan. Запустить?»

### 1.2. Предложение списка

Покажи список типовых пробелов, **убрав из него то, что уже есть в каталоге**:

```text
Что обычно добавляют после baseline:

1. Table       — список с заголовками, плотностями, состояниями
2. Tabs        — переключение между разделами на одном экране
3. Tooltip     — короткая подсказка над элементом
4. Dropdown    — триггер плюс панель опций или меню
5. Select      — поле выбора одного значения из списка
6. Toast       — всплывающее уведомление (success / warning / error / info)
7. Breadcrumb  — путь по иерархии страниц
8. Pagination  — постраничная навигация по списку
```

Спроси:

> Какие из этих добавляем? Можно убрать ненужные, добавить своё (имя + одна строка назначения). Бери только то, чего реально не хватает на экранах — UI-кит лучше держать компактным.

Если есть экраны или задачи в `workspace/product/tasks/`, можно подсказать, какие компоненты там встречаются, но решение за пользователем.

### 1.3. Чек на дубликаты

Для каждого компонента из итогового набора, особенно не из списка:
- точное или близкое совпадение имени в `components.md` (`Select` ↔ `Dropdown`, `Toast` ↔ `Alert`/`Notification`, `Tabs` ↔ `SegmentedControl`);
- совпадение по назначению.

Если близкий есть — спроси: «В каталоге уже есть X (назначение). Расширяем его варианты отдельной задачей или это действительно другой компонент?»
Зафиксируй итоговый набор.

---

## Фаза 2. Markdown-описания

Собери описания всех компонентов в одном файле `workspace/product/tasks/<TASK-ID>/analysis/grow-ui-kit-draft.md` по шаблону из [output-template.md](output-template.md). Это черновик: после апрува и отрисовки содержимое уходит в `ds/components.md` строками.

Правила описаний:
- **Минимальная матрица вариантов:** 2–3 варианта на компонент. Не плодить шесть variants со всеми состояниями — write-tools на больших матрицах сбоят.
- **Отступы** — только из шкалы `space-*` в `foundation.md`.
- **Привязки** — только к существующим Semantic Variables и TextStyles. Если нужного нет — отметь это в описании как блокер и предложи сначала добавить токен / стиль по правилам контракта.
- **Вложенные компоненты** — если внутри используется существующий компонент ДС (кнопка в Toast, иконка в Select), указывай его как инстанс по Node ID из `components.md`, а не перерисовывай.

Покажи все описания разом. Дождись апрува по каждому компоненту. Правка по одному — внеси и переспроси только по нему.
**До апрува никаких MCP-вызовов.**

---

## Фаза 3. Отрисовка через `use_figma`

### 3.0. Получение файла

- Возьми ссылку на Figma-файл из `ds/CONTRACT.md` или `ds/source.md`. Если её там нет, спроси:
  > Пришли ссылку на тот же Figma Design-файл, где лежит секция «Foundation». Новый блок «UI Kit — extended» поставим рядом.
- Извлеки `fileKey` (`figma.com/design/:fileKey/...`; для веток бери `branchKey`).
- Вызови `create_session` с `fileKey`, загрузи скилл `figma-use` и только после этого вызывай `use_figma`.

### 3.1. Подготовительное чтение

Один read-only вызов: найти секцию «Foundation» (и «UI Kit — extended», если это повторный прогон), получить Variables и TextStyles.

```js
const norm = s => s.replace(/\s*\/\s*/g, "/");
const page = figma.currentPage;
const foundation = page.findOne(n => n.type === "SECTION" && n.name === "Foundation");
const ext = page.findOne(n => n.type === "SECTION" && n.name === "UI Kit — extended");
const vars = await figma.variables.getLocalVariablesAsync();
const styles = await figma.getLocalTextStylesAsync();
return {
  foundation: foundation && { id: foundation.id, x: foundation.x, y: foundation.y, w: foundation.width },
  extended: ext && { id: ext.id, children: ext.children.map(c => c.name) },
  variables: vars.map(v => v.name),
  textStyles: styles.map(s => norm(s.name)),
};
```

Сверь со списком привязок из апрувнутых описаний. Если чего-то не хватает — остановись и вернись к пользователю.

### 3.2. Секция «UI Kit — extended»

Если секция уже есть (повторный прогон) — используй её, не создавай дубль. Если нет — создай справа от «Foundation» с зазором 80:

```js
const ext = figma.createSection();
ext.name = "UI Kit — extended";
ext.x = foundation.x + foundation.width + 80;
ext.y = foundation.y;
```

Внутри — один wrapping-фрейм `Components` (`layoutMode = "HORIZONTAL"`, `layoutWrap = "WRAP"`, `itemSpacing` и `counterAxisSpacing` привязаны к `space-8`). Каждый компонент — плитка: вертикальный Auto Layout с лейблом `DS/Label/xs` (цвет `text-muted`) над ComponentSet. Фон секции — `surface-subtle`, как у «Foundation».

### 3.3. Пакетная отрисовка

Группируй компоненты по 2–3 на один мутирующий вызов. Все 6–8 одним вызовом не делай: на длинных пакетах отваливаются привязки к Variables и разваливается Auto Layout.

#### Правила отрисовки

1. **Настоящие `ComponentNode` / `ComponentSetNode`.** Через `figma.createComponent()` или `figma.createComponentFromNode(frame)`. Варианты — через `figma.combineAsVariants([...], parent)`, имена до объединения в формате `Property=value` (`Density=compact`, `State=hover`). Фрейм-«компоненты» не видны в Assets и непригодны для `createInstance()`.

2. **Типографика — только существующие TextStyles** (`DS/…` из ds_baseline или стили ДС после ds_scan). Поиск через `getLocalTextStylesAsync()` с нормализацией пробелов вокруг `/`, применение через `await text.setTextStyleIdAsync(style.id)`. Перед этим `figma.loadFontAsync(style.fontName)`. **Новые TextStyles не создаются.** Прибивать `fontSize` / `fontName` запрещено. Цвет текста — отдельно, через `text.fills` с биндингом к Semantic (`text-default`, `text-muted`, …).

3. **Все fills и strokes — через Variables:**
   ```js
   const v = name => vars.find(x => x.name === name);
   const paint = figma.variables.setBoundVariableForPaint(
     { type: "SOLID", color: { r: 0, g: 0, b: 0 } }, "color", v("surface-default")
   );
   row.fills = [paint];
   ```
   Тени — только через существующие EffectStyle.

4. **Радиусы — через Variables, каждый угол отдельно** (`topLeftRadius`, `topRightRadius`, `bottomLeftRadius`, `bottomRightRadius`). **Отступы и `itemSpacing`** — `node.setBoundVariable("paddingLeft", v("space-4"))` и т.д., без прибитых чисел.

5. **Auto Layout везде.** Любой компонент, любая плитка, любая секция.

6. **Вложенные компоненты ДС — инстансами.** Кнопку внутри Toast или Pagination берём `createInstance()` из существующего компонента по Node ID, не рисуем заново.

7. **Имена латиницей.** ComponentSet — короткое (`Table`, `Tabs`, `Toast`); слои внутри — по смыслу (`Header`, `Row`, `Cell`), не `Frame 234`.

8. **Описание компонента** — `componentSet.description` = строка «Назначение» из апрувнутого описания.

9. **Никакого `throw` в конце мутирующего вызова.** `use_figma` исполняет вызов как транзакцию: `throw` откатывает все мутации (`createComponent`, `combineAsVariants`, `setTextStyleIdAsync`, `setBoundVariable*`). Отчёт — отдельным read-only вызовом; статус можно положить в `node.name` или `setSharedPluginData`. Симптом нарушения: лог говорит «applied=N», а в файле ничего не изменилось.

### 3.4. Проверка после каждого пакета

Отдельным read-only вызовом:

1. `findOne(n => (n.type === "COMPONENT_SET" || n.type === "COMPONENT") && n.name === "<Имя>")` внутри «UI Kit — extended» — узел нужного типа. `null` или `FRAME` — компонент не создан, исправь до следующего пакета.
2. Все text-узлы внутри компонента имеют непустой `textStyleId`. Пустые — `setTextStyleIdAsync` откатился или не применился.
3. Ни одного SOLID-fill или stroke без `boundVariables.color`.
4. `itemSpacing` и `padding*` либо привязаны к `space-*` (`boundVariables`), либо равны 0.
5. `get_screenshot` по секции «UI Kit — extended» — показать пользователю.

Запиши фактические Node ID созданных компонентов.

---

## Фаза 4. Каталог и аудит

### 4.1. Дописать `ds/components.md`

Каждый новый компонент — отдельной записью в разделе по смыслу (Действия / Ввод / Контейнеры / Навигация / Фидбэк / Данные) в формате каталога (шаблон в [output-template.md](output-template.md)), с фактическим Node ID. Существующие записи не трогаем. Если раздела нет — создай его.

Обнови шапку файла: количество наборов и дату расширения. Если компонент с таким именем уже был — пометь конфликт и спроси: переименовать новый или убрать старую запись.

Покажи пользователю добавленные записи.

### 4.2. Аудит консистентности

Один read-only вызов `use_figma` по всем компонентам секции «UI Kit — extended», сверка с правилами, по которым собрана «Foundation»:

1. **Типографика** — каждый text-узел привязан к TextStyle ДС; нет прибитых `fontSize` / `fontName` вне стиля.
2. **Цвета** — все fills / strokes привязаны к Semantic Variables (не к Primitive напрямую и не прибитый hex).
3. **Отступы и радиусы** — значения из шкалы `space-*` / `radius-*`, нет 14, 18, 22 и других не-токенных чисел.
4. **Структура** — тип `COMPONENT_SET` / `COMPONENT`, Auto Layout у корня и вложенных контейнеров, имена слоёв латиницей.

```js
const ext = figma.currentPage.findOne(n => n.type === "SECTION" && n.name === "UI Kit — extended");
const vars = await figma.variables.getLocalVariablesAsync();
const collOf = Object.fromEntries(
  (await figma.variables.getLocalVariableCollectionsAsync())
    .flatMap(c => c.variableIds.map(id => [id, c.name]))
);
const report = [];
for (const comp of ext.findAll(n => n.type === "COMPONENT_SET" || (n.type === "COMPONENT" && n.parent.type !== "COMPONENT_SET"))) {
  const issues = [];
  comp.findAll(() => true).concat(comp).forEach(n => {
    if (n.type === "TEXT" && !n.textStyleId) issues.push(`text без стиля: ${n.name}`);
    for (const key of ["fills", "strokes"]) {
      const paints = Array.isArray(n[key]) ? n[key] : [];
      paints.forEach(p => {
        if (p.type !== "SOLID") return;
        const id = p.boundVariables?.color?.id;
        if (!id) issues.push(`${key} прибит: ${n.name}`);
        else if (collOf[id] !== "Semantic") issues.push(`${key} не Semantic: ${n.name}`);
      });
    }
    if ("layoutMode" in n && n.layoutMode !== "NONE") {
      for (const k of ["itemSpacing", "paddingLeft", "paddingRight", "paddingTop", "paddingBottom"]) {
        if (n[k] && !n.boundVariables?.[k]) issues.push(`${k}=${n[k]} без токена: ${n.name}`);
      }
    }
  });
  report.push({ name: comp.name, id: comp.id, ok: issues.length === 0, issues });
}
return report;
```

Имя коллекции Semantic подставь фактическое (из `foundation.md`). Если ДС после ds_scan без отдельной Semantic-коллекции — проверяй только факт привязки к Variable.

Покажи отчёт построчно: компонент → ✓ или конкретная проблема, и на каждое расхождение — короткое предложение правки. Расхождения исправляются через `use_figma` (переотрисовка или точечный rebind), затем аудит повторяется. «Допишу руками потом» не принимается.

### 4.3. Финальный показ

По шаблону «Финальный отчёт» из [output-template.md](output-template.md).

---

## Синхронизация Figma ↔ md

- Структурные правки — сначала в `ds/components.md` / описаниях, затем перезапуск фазы 3 для этого компонента.
- Если компонент поправили руками в Figma — обнови запись в `components.md`. Расхождение между Figma и индексом ломает последующие сборки экранов.
- Черновик `analysis/grow-ui-kit-draft.md` после успешного прогона можно удалить, но только по согласию пользователя.
