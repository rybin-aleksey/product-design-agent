# Методология: аудит экранов

Фазы идут по порядку (гейты — в [checklist.md](checklist.md)).
`ds/` — `workspace/product/ds/`, `tasks/` — `workspace/product/tasks/`.

**Про инструмент:** `use_figma` — для чтения (и записи только в `--fix`); `get_screenshot` — для показа находок. Перед первым вызовом — `create_session` с `fileKey` и загрузка скилла `figma-use`.

---

## Фаза 0. Входы

### 0.1. Markdown

Прочитай:
- `ds/components.md` — имена, Node ID, матрицы вариантов, `used in`, `Status`;
- `ds/foundation.md` — Semantic и Primitive, тени;
- все composition maps `tasks/*/artifacts/design/screens/*.md` — по строке «Final Screen Node ID» сопоставляются с фреймами, разделы «Edge cases» и «осознанно пропущены»;
- последний отчёт `tasks/*/analysis/screens-audit-*.md` (если есть) — решения прошлых прогонов.

Это «правда» по версии markdown; дальше она сверяется с Figma.

### 0.2. Figma

Один read-only вызов:

```js
const page = figma.currentPage;
const section = page.findOne(n => n.type === "SECTION" && n.name === "Screens");
if (!section) return { error: "no Screens section" };

const vars = await figma.variables.getLocalVariablesAsync();
const cols = await figma.variables.getLocalVariableCollectionsAsync();
const colName = Object.fromEntries(cols.map(c => [c.id, c.name]));

return {
  screens: section.children
    .filter(n => n.type === "FRAME" && n.name.startsWith("Screen/"))
    .map(n => ({ name: n.name, id: n.id })),
  variables: vars.map(v => ({ id: v.id, name: v.name, collection: colName[v.variableCollectionId], type: v.resolvedType })),
  effectStyles: (await figma.getLocalEffectStylesAsync()).map(s => s.name),
  textStyles: (await figma.getLocalTextStylesAsync()).map(s => s.name),
};
```

- Секции «Screens» нет → «Нечего аудировать: сначала собери экраны через final-screens». Прогон закрыт.
- Компоненты ДС берутся по Node ID из `components.md` (`getNodeByIdAsync`), а не поиском по секции «Foundation»: часть живёт в «UI Kit — extended» или в файле ДС после ds_scan. Для каждого — тип, `componentPropertyDefinitions`, размер дефолтного варианта, fills.
- Если ДС подключена из другого файла (ds_scan), Variables и компоненты — удалённые: сопоставляй по `key` и именам, а не по локальным id. Пометь это в отчёте.
- Экран есть в Figma, но нет composition map (или наоборот) — это находка раздела 0 «Синхронизация».

---

## Фаза 1. Шесть проверок

Один read-only вызов на экран; возвращаются только находки. Проверки 3 и 4 считаются по каталогу после обхода всех экранов.

### Обход экрана

```js
const screen = await figma.getNodeByIdAsync("<screenId>");
const findings = [];
const instances = [];          // для проверок 4 и 5
const path = n => { const p = []; for (let x = n; x && x !== screen; x = x.parent) p.unshift(x.name); return p.join(" / "); };

const visit = async (n, insideInstance) => {
  // Проверка 1: явный след детача
  if (!insideInstance && (n.type === "FRAME" || n.type === "GROUP") && n.detachedInfo) {
    findings.push({ check: 1, node: n.id, path: path(n), detachedFrom: n.detachedInfo, confidence: "high" });
  }
  // Проверка 2: прибитые цвета и эффекты
  for (const key of ["fills", "strokes"]) {
    const paints = Array.isArray(n[key]) ? n[key] : [];
    paints.forEach((p, i) => {
      if (p.type === "SOLID" && p.visible !== false && !p.boundVariables?.color) {
        findings.push({ check: 2, node: n.id, path: path(n), key, index: i, color: p.color, opacity: p.opacity, insideInstance });
      }
    });
  }
  if (Array.isArray(n.effects) && n.effects.length && !n.effectStyleId) {
    findings.push({ check: 2, node: n.id, path: path(n), key: "effects", insideInstance });
  }
  if (n.type === "TEXT" && !insideInstance && !n.textStyleId) {
    findings.push({ check: 2, node: n.id, path: path(n), key: "textStyle" });
  }
  // Instance: запоминаем мастер и варианты, внутрь идём только за override
  if (n.type === "INSTANCE") {
    const main = await n.getMainComponentAsync();
    instances.push({
      node: n.id, path: path(n),
      main: main?.id, set: main?.parent?.type === "COMPONENT_SET" ? main.parent.id : main?.id,
      setName: main?.parent?.type === "COMPONENT_SET" ? main.parent.name : main?.name,
      variant: n.variantProperties,
    });
  }
  if ("children" in n) for (const c of n.children) await visit(c, insideInstance || n.type === "INSTANCE");
};
await visit(screen, false);
return { screen: screen.name, findings, instances, candidates: /* см. проверку 1 */ [] };
```

Внутри инстансов SOLID без привязки — это override (мастер привязан) или дефект мастера: проверь мастер отдельно и пометь, где источник.

### Проверка 1. Отвязанные элементы вместо instance

- **Высокая уверенность:** `detachedInfo` у фрейма — его детачнули от компонента; `detachedInfo` указывает, от какого.
- **Эвристика** для фреймов вне инстансов без `detachedInfo`: совпадение 2+ признаков с мастером из каталога —
  размер ±10 px по обеим осям; fill совпадает с Semantic, который использует мастер; анатомия (текст + иконка, две строки текста, прямоугольник + текст); имя похоже на имя компонента.
  2 признака — `medium`, 3+ — `high`, остальное в отчёт не идёт.
- Предлагаемый компонент — Set + значения вариантов, ближайшие по fill и размеру.

### Проверка 2. Прибитые значения без Variable

- Цвет без привязки сверь с Semantic (значение резолвится через alias в режиме по умолчанию): точное совпадение — `high`, расхождение ≤ 5 единиц по каналу RGB (0–255) — `medium`, иначе — «нет токена», предложение: ближайший Semantic или новый токен через Foundation.
- Если совпало несколько Semantic (например, `surface-default` и `text-on-action` оба белые) — выбирай по роли: fill фрейма → `surface-*`, fill текста → `text-*`, stroke → `border-*`; иначе спрашивай.
- Совпадение только с Primitive — находка «привязать к Semantic».
- Эффекты без EffectStyle и тексты без TextStyle — в тот же раздел, отдельными пунктами.

### Проверка 3. Дубликаты компонентов

По каталогу и мастерам:
- похожие имена после нормализации (`Card/Stats` ↔ `StatCard`, `PrimaryButton` ↔ `Button` Type=primary);
- похожая анатомия (одинаковая последовательность типов детей дефолтного варианта) и одинаковые Semantic-привязки;
- похожее назначение в `components.md`.

Записывается как подозрение с причиной и оценкой сходства. Разные смыслы (Card Type=default vs Type=interactive) — не дубликат.

### Проверка 4. Неиспользуемые компоненты

По собранным `instances` всех экранов против каталога:
- `unused` — ни одного инстанса и пустой `used in`;
- `stale` — в `used in` записан экран, а инстансов на нём нет;
- `unrecorded` — инстансы есть, а экрана нет в `used in`.

Компоненты, используемые только внутри других компонентов (Badge в ServerCard), не считаются unused: проверь вложенность в мастерах.

### Проверка 5. Расхождения в состояниях

Для каждого Set — какие значения осей реально встречаются в `variant` инстансов:
- `unused-variants` — значения есть в Set, на экранах не встречаются. Норма для hover / pressed / focus (они интерактивные и на статичных макетах не видны) — так и пометь; для disabled / loading / error — сигнал «забыли применить или избыточно».
- `custom-detached` — детачнутый фрейм из проверки 1, который отличается от всех вариантов мастера (другой fill / opacity / stroke) — либо в ДС не хватает варианта (→ design-system-variants), либо случайный детач.
- `matrix-mismatch` — оси / значения Set в Figma не совпадают со строкой «Варианты» в `components.md`.

### Проверка 6. Покрытие состояний (edge cases)

Для каждой секции экрана по composition map (и по фреймам секций в Figma) определи тип блока и сверь с матрицей:

| Тип блока | Обязательные состояния |
|---|---|
| Список / таблица / лента | с данными, пусто, загрузка, ошибка, переполнение |
| Форма | по умолчанию, валидные поля, ошибки полей, отправка, успех, ошибка отправки |
| Карточка контента | по умолчанию, загрузка, ошибка, удалённый элемент |
| Авто-обновляемая зона | первое появление, загрузка, устаревшие, свежие |
| Доступ | по умолчанию, запрещён, заблокировано, требуется вход, требуется оплата |

Состояние считается заложенным, если есть: скрытый сосед (`visible: false`) с понятным именем, вариант компонента в инстансе, или запись в разделе «Edge cases» composition map. Пометки «осознанно пропущены» из карты и решения прошлых отчётов — исключаются из находок.

Контекст: hero, маркетинговые секции, статичный контент — состояний не требуют; формы в админке — все. Сомнительный тип блока — укажи предположение в находке.

---

## Фаза 2. Отчёт

Сформируй отчёт по шаблону из [output-template.md](output-template.md): шапка с числами, раздел 0 (синхронизация md ↔ Figma, если есть расхождения), затем шесть разделов. В каждой находке: экран / путь узла, суть, уверенность, предлагаемое действие и пометка «автофикс возможен» или «не автофикс → кому передать».

Порядок важности: разделы 1 и 5 («почти такие же» элементы) и 6 (edge cases) — первыми в сводке; 2–4 — поддерживающие.
Для 3–5 спорных находок можно показать `get_screenshot` узла.

Затем по каждой находке спроси решение:
- **Чинить** — автофикс (только в `--fix`; без флага — предложи перезапустить с `--fix` или передать задачу);
- **Пропустить** — оставить, зафиксировать как осознанное решение;
- **Не решать сейчас** — остаётся открытой в отчёте.

Находки одного типа можно предложить решить группой («все 6 прибитых `#FAFAFA` → `surface-subtle`?»), если пользователь согласен решать так.

---

## Фаза 3. Автофиксы (только `--fix`, только «чинить»)

Каждый фикс — мутирующий вызов без `throw` в конце; после пакета фиксов одного раздела — read-only проверка.

### Проверка 1: отвязанный → instance

```js
const target = await figma.getNodeByIdAsync("<nodeId>");
const set = await figma.getNodeByIdAsync("<setId>");
const inst = set.type === "COMPONENT_SET" ? set.defaultVariant.createInstance() : set.createInstance();
inst.setProperties({ /* варианты из находки */ });

const parent = target.parent;
parent.insertChild(parent.children.indexOf(target), inst);
if (parent.layoutMode && parent.layoutMode !== "NONE") {
  inst.layoutSizingHorizontal = target.layoutSizingHorizontal;
  inst.layoutSizingVertical = target.layoutSizingVertical;
} else {
  inst.x = target.x; inst.y = target.y;
}
// Перенос текстов: TEXT-свойства инстанса через setProperties, иначе по совпадению имён текстовых слоёв
inst.name = target.name;
target.remove();
```

Перед заменой сравни тексты старого узла и нового инстанса: если текст не переносится однозначно — покажи пользователю и спроси. Размер инстанса не подгоняй `resize`, если родитель в Auto Layout.

### Проверка 2: прибитый цвет → Variable

```js
const node = await figma.getNodeByIdAsync("<nodeId>");
const variable = (await figma.variables.getLocalVariablesAsync()).find(v => v.name === "<semantic>");
const paints = [...node["<fills|strokes>"]];
paints[<index>] = figma.variables.setBoundVariableForPaint(paints[<index>], "color", variable);
node["<fills|strokes>"] = paints;
```

Эффекты → `setEffectStyleIdAsync`, тексты → `setTextStyleIdAsync` (после `loadFontAsync`). Если источник — мастер, а не экран, фикс делается в мастере только для своей / Community ДС и с отдельным апрувом; для корпоративной — передаётся в команду ДС.

### Проверка 4: каталог

Не Figma-операция. В `ds/components.md`: исправить `used in` (`stale`, `unrecorded`) или добавить `**Status:** не используется на экранах (аудит <дата>), кандидат на удаление`. Сам компонент не удаляется.

### Проверки 3, 5, 6

Не автофиксы:
- 3 — объединение компонентов → отдельная задача (решение дизайнера);
- 5 — недостающие варианты → [design-system-variants](../design-system-variants/SKILL.md); `matrix-mismatch` — исправление строки в `components.md` с апрувом;
- 6 — недостающие состояния → [final-screens](../final-screens/SKILL.md) с брифом «расширь состояния на секции <name>: <список>».

---

## Фаза 4. Запись

### 4.1. Отчёт

`tasks/<TASK-ID>/analysis/screens-audit-<YYYY-MM-DD>.md` (если файл с этой датой есть — суффикс `-2`). Содержит отчёт, решение по каждой находке, результат автофиксов и «было / стало» относительно прошлого отчёта.

### 4.2. `ds/components.md`

Только по решениям пользователя: `used in` (после фиксов 1 и 4), `Status`, исправленная строка вариантов.

### 4.3. Composition maps

По каждому решению «оставить осознанно» в проверке 6 — раздел «Edge cases — осознанно пропущены» в карте экрана с причиной и датой. Следующий аудит эти пункты пропускает.

---

## Фаза 5. Финальный отчёт

По шаблону из [output-template.md](output-template.md): что починено, что помечено, что передано в другие скиллы, что оставлено осознанно, что открыто.

---

## Частота

После каждого экрана через final-screens — нормально. Повторный прогон без изменений в Figma бессмысленен: сначала посмотри, что изменилось с прошлого отчёта.
