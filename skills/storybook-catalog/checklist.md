# Чек-лист: каталог в Storybook

`code/` — `workspace/product/code/`, `ds/` — `workspace/product/ds/`.

## Перед стартом

- [ ] Пользователь явно запустил storybook.
- [ ] React-база найдена: `src/components/`, `src/tokens/`, проект ставится.
- [ ] Node.js 18+ и npm есть.
- [ ] `ds/components.md`, `ds/foundation.md`, `ds/CONTRACT.md` прочитаны.

## Фаза 0–1

- [ ] Таблица «components.md × база × stories» составлена, расхождения показаны.
- [ ] Init запущен только при отсутствии `.storybook/`, с `--yes`.
- [ ] Примеры init удалены.
- [ ] Токены и шрифт подключены в preview; autodocs в конфиге не тронут.

## Self-check (перед финалом)

- [ ] На каждый компонент из `src/components/` есть `.stories.tsx` со всеми вариантами и состояниями из `components.md`. Отсутствующие перечислены (для полного каталога — пусто).
- [ ] У каждого компонента `tags: ['autodocs']` и полные `argTypes` → страница с контролами и Show code / Copy code.
- [ ] У каждого компонента есть `AllVariants`.
- [ ] Foundation показывает Primitive и Semantic (значения из `src/tokens/`, не хардкод) и типографику.
- [ ] Переходы видны живьём минимум на Button и Input; на токенах; `prefers-reduced-motion` учтён.
- [ ] Песочницы собраны из реальных компонентов и реагируют.
- [ ] Сложные кейсы — добавлены по согласию или пропущены по ответу.
- [ ] `npm run storybook -- --ci` стартует, `build-storybook` проходит без ошибок.
- [ ] Новые зависимости — только согласованные; ничего не закоммичено без указания.

## Частые ошибки

| Симптом | Причина | Что делать |
|---|---|---|
| Init завис | Интерактивные вопросы | `npx storybook@latest init --yes` |
| Сломался конфиг после повторного запуска | Init запущен поверх `.storybook/` | Init только при отсутствии `.storybook/` |
| Компоненты без темы и шрифта | Токены не подключены в preview | Импорт `src/tokens/*.css` и шрифта в `.storybook/preview` |
| Нет таблицы пропсов и Copy code | Нет `tags: ['autodocs']` | Тег в `meta` каждого компонента |
| Контролы — текстовые поля для перечислений | Нет `argTypes` с `options` | `inline-radio` / `select` + `options` |
| В каталоге чужие примеры | Не удалён `src/stories/` от init | Удалить |
| Каталог расходится с приложением | Stories рисуют разметку сами | Только `import` из `src/components` |
| Процесс Storybook блокирует работу | Запущен в foreground | Запуск в фоне с `--ci` |
