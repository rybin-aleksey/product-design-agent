# Шаблоны выходных данных

`ds/` — `workspace/product/ds/`, `tasks/` — `workspace/product/tasks/`.

| Файл | Фаза | Назначение |
|---|---|---|
| `tasks/<TASK-ID>/analysis/screens-audit-<YYYY-MM-DD>.md` | 4 | Полный отчёт прогона с решениями |
| `ds/components.md` | 4 | `used in`, `Status`, строка вариантов — по решениям |
| `tasks/*/artifacts/design/screens/<name>.md` | 4 | «Edge cases — осознанно пропущены» |

---

## Отчёт (фаза 2)

```markdown
# Screens audit: <YYYY-MM-DD>

**Режим:** audit only / --fix
**Экраны:** <N> (Dashboard, Files, Settings, Landing)
**Находок:** <M> (high: <a>, medium: <b>, подозрений: <c>)
**Прошлый аудит:** <дата / нет>; закрыто с тех пор: <k>, новых: <l>

## 0. Синхронизация md ↔ Figma
- Screen/Landing — нет composition map
- <если расхождений нет — раздел опускается>

## 1. Отвязанные элементы вместо instance
- [Landing / Hero / Button-imitator] — детачнут от Button (high). Заменить на Button Type=primary, Size=lg. Автофикс возможен.

## 2. Прибитые значения без Variable
- [Settings / FormBackground] fill `#FAFAFA` = `surface-subtle` (high). Автофикс возможен.
- [Dashboard / Stats Row / Label] текст без TextStyle → DS/Body/sm (medium). Автофикс возможен.

## 3. Дубликаты компонентов (подозрения)
- Card/Stats ↔ StatCard — сходство 0.85: похожие имена, одинаковая анатомия (title + value + badge). Решение за дизайнером.

## 4. Неиспользуемые компоненты
- IconButton — unused: нет на экранах, `used in` пуст. Пометить «кандидат на удаление» / оставить?
- Card — stale: в `used in` есть TASK-001/screens/files, на экране инстансов нет. Исправить запись?

## 5. Расхождения в состояниях
- Button — State: disabled, loading не используются ни на одном экране (hover — норма). Применить или избыточно?
- Input — matrix-mismatch: в Figma State: default | error | focus, в components.md — default | error.

## 6. Покрытие состояний
- Files / Files Table (список): пусто ✓, загрузка ✗, ошибка ✗, переполнение ✗
- Settings / Form (форма): ошибки полей ✗, отправка ✗, успех ✗, ошибка отправки ✗
- Landing — маркетинговая страница, состояния не требуются

## Что дальше
По каждой находке: чинить / пропустить / не решать сейчас.
```

Пустой раздел — «Не найдено».

---

## Пометка в composition map (фаза 4.3)

```markdown
## Edge cases — осознанно пропущены
- Files Table → переполнение: пропущено осознанно, короткие имена файлов (аудит <YYYY-MM-DD>)
- ActivityFeed → ошибка загрузки: в следующей итерации (аудит <YYYY-MM-DD>)
```

## Пометка в `ds/components.md` (фаза 4.2)

```markdown
### IconButton
- …
- **Used in:** —
- **Status:** не используется на экранах (аудит <YYYY-MM-DD>), кандидат на удаление
```

## Бриф для final-screens (из проверки 6)

```text
final_screens <ScreenName>: расширь состояния на секции <Section> — <загрузка, ошибка>.
Источник: screens-audit <YYYY-MM-DD>.
```

---

## Финальный отчёт (фаза 5)

```text
Аудит завершён (<режим>).

Экранов: <N>, находок: <M>

Починено автофиксом: <k>
- Landing / Button-imitator → instance Button Type=primary, Size=lg
- Settings / FormBackground → surface-subtle

Помечено в каталоге: <k>
- IconButton → «кандидат на удаление»

Передано дальше: <k>
- final-screens: Files Table — загрузка, ошибка
- design-system-variants: <если есть>

Оставлено осознанно: <k> (пометки в composition maps)
Открыто («не решать сейчас»): <k>

Отчёт: workspace/product/tasks/<TASK-ID>/analysis/screens-audit-<YYYY-MM-DD>.md
Обновлены: <ds/components.md, composition maps / ничего>
Figma: <изменена N фиксами / не тронута>
```
