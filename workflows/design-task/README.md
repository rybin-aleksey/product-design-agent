# Design task

Workflow для выполнения отдельной дизайн-задачи продукта.

## Как выбрать ветку

1. Определи задачу: `workspace/product/tasks/<TASK-ID>/`.
2. Проверь, есть ли у продукта ДС: если существует `workspace/product/ds/CONTRACT.md`, прочитай контракт до начала работы с интерфейсом.
3. Выбери ветку по запросу.

| Запрос | Ветка |
|---|---|
| «Создай / поставь дизайн-систему с нуля», «заведи токены и UI-кит», у продукта нет ДС | [Создание дизайн-системы с 0](#ветка-создание-дизайн-системы-с-0) |
| «Подключи / проиндексируй нашу ДС», «работаем по корпоративной ДС», «взяли ДС из Community», ДС в Figma уже есть | [Подключение существующей дизайн-системы](#ветка-подключение-существующей-дизайн-системы) |
| «Добавь в ДС таблицу / табы / тосты», «расширь UI-кит», ДС есть, а компонентов не хватает | [Расширение UI-кита](#ветка-расширение-ui-кита) |
| «Добавь hover / disabled / focus / loading», «добавь размеры», «доделай состояния компонентов», компонент в ДС уже есть | [Расширение вариантов и состояний](#ветка-расширение-вариантов-и-состояний) |
| «Собери экран», «сделай финальный макет по wireframe», «переведи wireframes в UI», ДС подключена | [Сборка финальных экранов](#ветка-сборка-финальных-экранов) |
| «Проверь экраны», «аудит перед разработкой», «найди расхождения с ДС», экраны собраны | [Аудит экранов](#ветка-аудит-экранов) |
| `storybook`, «собери каталог компонентов в Storybook», React-база собрана | [Каталог компонентов в Storybook](#ветка-каталог-компонентов-в-storybook) |
| `screens` + ссылка на фрейм, «собери экран в коде / в React» | [Сборка экранов в коде](#ветка-сборка-экранов-в-коде) |
| `wire`, «оживи экраны», «сделай кликабельным», статичные экраны собраны | [Оживление экранов](#ветка-оживление-экранов) |
| `handoff`, «передай проект разработке», «залей на GitHub» | [Передача разработке](#ветка-передача-разработке) |
| `deploy`, «опубликуй прототип», «нужна публичная ссылка» | [Публикация прототипа](#ветка-публикация-прототипа) |
| `dashboard`, «подключи дашборд юзер-тестов», «метрики юзер-теста», бандл `dashboard/` в проекте | [Юзер-тесты на прототипе](#ветка-юзер-тесты-на-прототипе) |

Все файлы дизайн-системы во всех ветках лежат в `workspace/product/ds/` — это общая папка продукта, а не папка задачи. Рабочие заметки прогона — в `workspace/product/tasks/<TASK-ID>/analysis/`.

## Ветка: Создание дизайн-системы с 0

Запускает директиву ds_baseline из скилла [skills/design-system-from-0](../../skills/design-system-from-0/SKILL.md).

**Условие входа:** у продукта нет действующей ДС, то есть нет `workspace/product/ds/CONTRACT.md`. Если ДС уже есть, ветка не запускается: расширение UI-кита и ребрендинг — отдельные задачи. Если ДС есть в Figma, но не проиндексирована — ветка «Подключение существующей дизайн-системы».

**Шаги:**

1. Прочитай [SKILL.md](../../skills/design-system-from-0/SKILL.md), затем [methodology.md](../../skills/design-system-from-0/methodology.md).
2. **Бриф.** Задай пять вопросов и сохрани `ds/brand-brief.md`.
3. **Foundation.** Собери `ds/foundation.md` и **дождись апрува**. До апрува никаких MCP-вызовов.
4. **Figma.** Получи ссылку на Figma-файл продукта, создай Variables и секцию «Foundation» с 7 базовыми компонентами, проверь результат по факту и запиши `ds/components.md`.
5. **Контракт.** Создай `ds/CONTRACT.md`. Корневые `CLAUDE.md` и `AGENTS.md` не меняй.
6. Сверься с [checklist.md](../../skills/design-system-from-0/checklist.md) и выдай финальный отчёт по [output-template.md](../../skills/design-system-from-0/output-template.md).

**Результат:** `workspace/product/ds/` (brand-brief, foundation, components, CONTRACT), а в Figma коллекции Primitive и Semantic и секция «Foundation».

**Ветка завершена**, только когда пройдена фаза 4 (контракт). Без неё ДС остаётся декорацией.

## Ветка: Подключение существующей дизайн-системы

Запускает директиву ds_scan из скилла [skills/design-system-scan](../../skills/design-system-scan/SKILL.md).

**Условие входа:** ДС уже собрана в Figma (своя, корпоративная или дублированная из Community), а её индекса в `workspace/product/ds/` нет или он устарел. Если ДС в Figma нет — ветка «Создание дизайн-системы с 0».

**Шаги:**

1. Прочитай [SKILL.md](../../skills/design-system-scan/SKILL.md), затем [methodology.md](../../skills/design-system-scan/methodology.md).
2. **Контекст.** Задай четыре вопроса, добейся ссылки именно на страницу Components и сохрани `ds/source.md`.
3. **Foundation.** Одним read-only вызовом прочитай Variables, собери `ds/foundation.md`, покажи пользователю и уточни неясные токены.
4. **Components.** Прочитай только верхний уровень страницы Components, собери `ds/components.md` с Node ID и пройди self-check.
5. **Показ.** Выдай сводку с замечаниями.
6. **Контракт.** Создай `ds/CONTRACT.md` с правилами под источник. Корневые `CLAUDE.md` и `AGENTS.md` не меняй.
7. Сверься с [checklist.md](../../skills/design-system-scan/checklist.md) и выдай финальный отчёт по [output-template.md](../../skills/design-system-scan/output-template.md).

**Результат:** `workspace/product/ds/` (source, foundation, components, CONTRACT). В Figma ничего не меняется.

**Ветка завершена**, только когда пройдена фаза 5 (контракт). Без неё индекс остаётся справочником.

## Ветка: Расширение UI-кита

Запускает директиву grow_ui_kit из скилла [skills/design-system-grow](../../skills/design-system-grow/SKILL.md).

**Условие входа:** у продукта есть ДС (`workspace/product/ds/CONTRACT.md` и непустой `components.md`), источник — ds_baseline, своя или дублированная Community. Если ДС нет — ветки выше. Если ДС корпоративная — недостающее оформляется локальным паттерном в `ds/patterns.md` по контракту.

**Шаги:**

1. Прочитай [SKILL.md](../../skills/design-system-grow/SKILL.md), затем [methodology.md](../../skills/design-system-grow/methodology.md).
2. **Каталог и выбор.** Прочитай `ds/components.md`, предложи типовые пробелы без уже существующих, проверь дубликаты и зафиксируй набор.
3. **Описания.** Собери md-описания в `tasks/<TASK-ID>/analysis/grow-ui-kit-draft.md` и **дождись апрува** по каждому компоненту. До апрува никаких MCP-вызовов.
4. **Figma.** В том же файле создай секцию «UI Kit — extended» рядом с «Foundation», отрисуй компоненты пакетами по 2–3 и проверь каждый пакет по факту.
5. **Каталог и аудит.** Допиши записи в `ds/components.md` с Node ID и проведи аудит консистентности с baseline.
6. Сверься с [checklist.md](../../skills/design-system-grow/checklist.md) и выдай финальный отчёт по [output-template.md](../../skills/design-system-grow/output-template.md).

**Результат:** дописанный `workspace/product/ds/components.md`, в Figma — секция «UI Kit — extended».

**Ветка завершена**, только когда аудит пройден без открытых расхождений.

## Ветка: Расширение вариантов и состояний

Запускает директиву component_variants из скилла [skills/design-system-variants](../../skills/design-system-variants/SKILL.md).

**Условие входа:** компонент уже есть в `workspace/product/ds/components.md` и в Figma, источник ДС — ds_baseline, своя или дублированная Community. Если компонента нет — ветка «Расширение UI-кита». Если ДС корпоративная — запрос в команду ДС по контракту.

**Режимы:** single — `component_variants <ComponentName>`; bulk — `component_variants` без аргумента, агент проходит по каталогу и по каждому неполному компоненту спрашивает, расширять ли.

**Шаги (для каждого компонента):**

1. Прочитай [SKILL.md](../../skills/design-system-variants/SKILL.md), затем [methodology.md](../../skills/design-system-variants/methodology.md).
2. **Read.** Найди запись в `ds/components.md`, прочитай компонент в Figma read-only, реши расхождения с пользователем.
3. **Brief.** Три вопроса: варианты, состояния, размеры. Дефолтов нет.
4. **md.** Посчитай новую матрицу (25+ ячеек — дробление), предложи Semantic-токены через alias, собери описание в `tasks/<TASK-ID>/analysis/component-variants-<name>.md` и **дождись апрува**. До апрува никаких мутирующих MCP-вызовов.
5. **Canvas.** Создай токены в Figma, при необходимости промоутни компонент в Set, дострой ячейки одним вызовом и проверь по факту.
6. **Каталог.** Обнови строку в `ds/components.md`, допиши токены в `ds/foundation.md`.
7. Сверься с [checklist.md](../../skills/design-system-variants/checklist.md) и выдай отчёт по [output-template.md](../../skills/design-system-variants/output-template.md).

**Результат:** расширенный Component Set в Figma, обновлённые `ds/components.md` и (при необходимости) `ds/foundation.md`.

## Ветка: Сборка финальных экранов

Запускает директиву final_screens из скилла [skills/final-screens](../../skills/final-screens/SKILL.md).

**Условие входа:** есть `workspace/product/ds/CONTRACT.md` и непустой `components.md`, в Figma — wireframe экрана в секции «Wireframes». Если ДС нет — ветки создания или подключения ДС.

**Режимы:** single — `final_screens <ScreenName>`; bulk — `final_screens` без аргумента, агент идёт по несобранным wireframes и по каждому спрашивает, собирать ли.

**Шаги (для каждого экрана):**

1. Прочитай [SKILL.md](../../skills/final-screens/SKILL.md), затем [methodology.md](../../skills/final-screens/methodology.md).
2. **Ресурсы.** Библиотека иконок и тени из Foundation.
3. **Read.** Wireframe read-only, каталог, сверка покрытия, анализ достаточности wireframe, повторяющиеся блоки — решения пользователя по каждому пункту.
4. **Composition map.** `tasks/<TASK-ID>/artifacts/design/screens/<name>.md` и **апрув**. До апрува никаких мутирующих MCP-вызовов.
5. **Вынос в UI-кит** (если решено): брифы и токены с апрувом, компоненты в «UI Kit — extended».
6. **Pre-canvas check** и **сборка** `Screen/<Name>` в секции «Screens» с проверкой по факту.
7. **Каталог.** `used in` в `ds/components.md`, индекс экранов задачи.
8. Сверься с [checklist.md](../../skills/final-screens/checklist.md) и выдай отчёт по [output-template.md](../../skills/final-screens/output-template.md).

**Результат:** фрейм `Screen/<Name>` в Figma, composition map и индекс в `artifacts/design/screens/` задачи, обновлённый `ds/components.md`.

После сборки рекомендуется контрольный прогон ветки «Аудит экранов».

## Ветка: Аудит экранов

Запускает директиву screens_audit из скилла [skills/screens-audit](../../skills/screens-audit/SKILL.md).

**Условие входа:** в Figma есть секция «Screens» с фреймами `Screen/<Name>`, есть `ds/components.md`. Если экранов нет — ветка «Сборка финальных экранов».

**Режимы:** `screens_audit` — только чтение и отчёт (по умолчанию); `screens_audit --fix` — после отчёта точечные автофиксы с апрувом на каждый.

**Шаги:**

1. Прочитай [SKILL.md](../../skills/screens-audit/SKILL.md), затем [methodology.md](../../skills/screens-audit/methodology.md).
2. **Входы.** Экраны из «Screens», компоненты по Node ID, Variables и стили, composition maps всех задач, прошлый отчёт.
3. **Шесть проверок** read-only, по одному вызову на экран.
4. **Отчёт** по шести разделам и решение пользователя по каждой находке.
5. **Автофиксы** — только в `--fix` и только на «чинить». Состояния передаются в final-screens, варианты — в design-system-variants.
6. **Запись.** Отчёт в `tasks/<TASK-ID>/analysis/screens-audit-<дата>.md`, решения — в `ds/components.md` и composition maps.
7. Сверься с [checklist.md](../../skills/screens-audit/checklist.md) и выдай финальный отчёт по [output-template.md](../../skills/screens-audit/output-template.md).

**Результат:** отчёт аудита с решениями; Figma меняется только по подтверждённым автофиксам.

## Ветка: Каталог компонентов в Storybook

Запускает директиву storybook из скилла [skills/storybook-catalog](../../skills/storybook-catalog/SKILL.md). **Только по явной команде пользователя** — это задача с кодом.

**Условие входа:** собрана React-база в `workspace/product/code/` (`src/components/` по именам из `ds/components.md`, `src/tokens/` с Primitive и Semantic), есть Node.js 18+. Если базы нет — её нужно собрать отдельной задачей (react_base; скилла в шаблоне пока нет).

**Шаги:**

1. Прочитай [SKILL.md](../../skills/storybook-catalog/SKILL.md), затем [methodology.md](../../skills/storybook-catalog/methodology.md).
2. **Контекст.** Сверь `ds/components.md` с `src/components/index.ts`, покажи расхождения.
3. **Установка.** `storybook init --yes`, только если нет `.storybook/`; удалить примеры; токены и шрифт в preview.
4. **Foundation, stories, переходы, песочницы** — по единому формату CSF3 + autodocs.
5. **Сложные кейсы** — только по согласию пользователя.
6. **Запуск** `npm run storybook -- --ci` в фоне, self-check по [checklist.md](../../skills/storybook-catalog/checklist.md), отчёт по [output-template.md](../../skills/storybook-catalog/output-template.md).

**Результат:** каталог в `workspace/product/code/` и адрес запущенного Storybook. Повторный запуск дополняет каталог.

## Ветка: Сборка экранов в коде

Запускает директиву screens из скилла [skills/react-screens](../../skills/react-screens/SKILL.md). **Только по явной команде пользователя со ссылкой на фрейм экрана** — это задача с кодом.

**Условие входа:** есть React-база в `workspace/product/code/`, дизайн экрана — отдельный фрейм в Figma (обычно `Screen/<Name>` из ветки «Сборка финальных экранов»).

**Шаги (на каждый экран):**

1. Прочитай [SKILL.md](../../skills/react-screens/SKILL.md), затем [methodology.md](../../skills/react-screens/methodology.md).
2. **Каркас.** Роутер, реестр `src/screens/registry.ts`, индекс «/» — завести, если нет (новая зависимость — с согласия).
3. **Дизайн.** Ссылка на фрейм; `get_metadata` → `get_design_context` по секциям → `get_screenshot`.
4. **Декомпозиция и сборка.** Компоненты базы и локальные блоки, отзывчивая раскладка, только токены, статичный контент из макета. Без логики и данных.
5. **Подключение.** Реестр, плитка на индексе, заметка экрана.
6. **Проверка** `npm run dev`, сверка с Figma, ширина и длинный контент; self-check по [checklist.md](../../skills/react-screens/checklist.md), отчёт по [output-template.md](../../skills/react-screens/output-template.md).

**Результат:** статичный экран в `src/screens/<name>/`, открывается плиткой с индекса «/». Повторный запуск добавляет новый экран или обновляет собранный.

## Ветка: Оживление экранов

Запускает директиву wire из скилла [skills/react-wire](../../skills/react-wire/SKILL.md). **Только по явной команде пользователя** — это задача с кодом.

**Условие входа:** статичные экраны собраны веткой «Сборка экранов в коде» и зарегистрированы в `src/screens/registry.ts`. Без бэкенда, деплоя и метрик.

**Шаги (на каждый запуск):**

1. Прочитай [SKILL.md](../../skills/react-wire/SKILL.md), затем [methodology.md](../../skills/react-wire/methodology.md).
2. **Скоуп.** Какие экраны / сценарий оживляем; edge cases из composition maps и screens-audit.
3. **Навигация** — переходы, «назад», активный пункт, 404.
4. **Мок-данные** — единый слой `src/data/`, экраны переводятся со статики на данные.
5. **Формы и поведение** — валидация, действие → результат; продуктовую логику, которой нет в макете, **задаёт пользователь**.
6. **Состояния и edge cases** — загрузка / успех / ошибка / пусто + границы и доменные края; дополнительные — по выбору пользователя. Анимации — по желанию.
7. **Проверка** — прокликать сценарий и сценарии из переключателя; self-check по [checklist.md](../../skills/react-wire/checklist.md), отчёт по [output-template.md](../../skills/react-wire/output-template.md).

**Результат:** кликабельный продукт на моке. Повторный запуск наращивает: ещё экран, поведение, анимации.

## Ветка: Передача разработке

Запускает директиву handoff из скилла [skills/github-handoff](../../skills/github-handoff/SKILL.md). **Только по явной команде пользователя**: команда разрешает один коммит и пуш в репозиторий для разработки, но не в `origin` продукта.

**Условие входа:** есть что передавать (обычно после ветки «Каталог компонентов в Storybook»), `git` в системе, аккаунт на GitHub.

**Шаги:**

1. Прочитай [SKILL.md](../../skills/github-handoff/SKILL.md), затем [methodology.md](../../skills/github-handoff/methodology.md).
2. **Шаги для пользователя.** Пустой приватный репозиторий; токен — только если нет менеджера учётных данных.
3. **Состав.** Что заливаем, `.gitignore`, проверка на секреты, **подтверждение** состава и адреса.
4. **Пуш** одним коммитом и одним `git push`.
5. **README** для разработчика и раздел «Репозиторий разработки» в `workspace/product/project.md`.
6. Self-check по [checklist.md](../../skills/github-handoff/checklist.md), отчёт со ссылкой по [output-template.md](../../skills/github-handoff/output-template.md).

**Результат:** HTTPS-ссылка на репозиторий для разработчика. Повторный запуск обновляет тот же репозиторий (`main` или отдельная ветка).

## Ветка: Публикация прототипа

Запускает директиву deploy из скилла [skills/deploy](../../skills/deploy/SKILL.md). **Только по явной команде пользователя**: команда разрешает публикацию выбранным путём, но не другие внешние действия.

**Условие входа:** прототип в `workspace/product/code/` работает локально (обычно после ветки «Оживление экранов»), `npm run build` проходит. Для сервиса — GitHub-репозиторий из ветки «Передача разработке»; для своего сервера — IP и SSH-доступ.

**Шаги:**

1. Прочитай [SKILL.md](../../skills/deploy/SKILL.md), затем [methodology.md](../../skills/deploy/methodology.md).
2. **Путь и подготовка.** Сервис или свой сервер; сборка, `base: '/'`, `.gitignore`, SPA-fallback.
3. **Сервис:** обновить репозиторий через github-handoff, браузерные шаги импорта, снять защиту Vercel, стабильная ссылка.
4. **Свой сервер:** вход по SSH-ключу, разведка read-only, **план и подтверждение**, nginx на свободном порту, заливка `dist`, firewall — не задевая VPN.
5. **Проверка:** прямой переход на внутренний экран — без 404.
6. **Запись** раздела «Публикация» в `workspace/product/project.md` без доступов; self-check по [checklist.md](../../skills/deploy/checklist.md), итог по [output-template.md](../../skills/deploy/output-template.md).

**Результат:** публичная ссылка на прототип. Повторный запуск обновляет публикацию тем же путём.

## Ветка: Юзер-тесты на прототипе

Запускает директиву dashboard из скилла [skills/ut-dashboard](../../skills/ut-dashboard/SKILL.md). **Только по явной команде пользователя**: изменения на сервере, в прототипе и передеплой.

**Условие входа:** прототип опубликован (ветка «Публикация прототипа»), бандл распакован в `workspace/product/dashboard/`, есть свой VPS с SSH-доступом.

**Шаги:**

1. Прочитай [SKILL.md](../../skills/ut-dashboard/SKILL.md), затем [methodology.md](../../skills/ut-dashboard/methodology.md).
2. **План теста.** README и код бандла; путь (http / https через ngrok); сценарии и гипотезы из материалов проекта → `tasks/<TASK-ID>/artifacts/validate/ut-plan.md`; **апрув сценариев и порогов**.
3. **Сбор** — служба на сервере (план и подтверждение, VPN не задевать); **адрес сбора** — порт наружу или ngrok-служба с постоянным адресом.
4. **Трекер** в прототипе (`data-endpoint`, `data-track`), **`config.js`**, гипотезы на сервер.
5. **Передеплой** через deploy, свой прогон по ссылке U-теста, первый раунд.
6. Self-check по [checklist.md](../../skills/ut-dashboard/checklist.md), итог по [output-template.md](../../skills/ut-dashboard/output-template.md).

**Результат:** живой прототип шлёт поведение, дашборд показывает метрики и вердикты по гипотезам; цикл «находки → правки агента → ретест».
