# Методология: передача на GitHub

`product/` — `workspace/product/`, `code/` — `workspace/product/code/`, `<scratch>` — scratchpad сессии.
Гейты — в [checklist.md](checklist.md). Тексты для пользователя — в [output-template.md](output-template.md).

---

## Фаза 0. Шаги и проверки

1. `git --version`.
2. Если в `product/project.md` уже есть раздел «Репозиторий разработки» — это повторный запуск, переходи к фазе 5.
3. Способ авторизации:
   - `git config --get credential.helper` → `manager` (Git Credential Manager, по умолчанию в Git for Windows) — при пуше откроется вход в GitHub в браузере, токен не нужен;
   - `gh auth status` → залогинен — тоже без токена;
   - иначе — токен (шаг ниже).
4. Выведи пользователю шаги (тексты в [output-template.md](output-template.md)):
   - **создать пустой приватный репозиторий** — без README, .gitignore и license;
   - **токен — только если нет менеджера учётных данных:** classic, scope `repo` (этого хватает для приватных репозиториев), срок 7–30 дней, скопировать один раз. Права шире `repo` и бессрочный токен не нужны: если токен утечёт, им можно сделать всё в аккаунте.
5. Попроси прислать ссылку на репозиторий (и токен, если он нужен). Токен не сохраняй и не повторяй в ответах. Предупреди: токен, присланный в чат, остаётся в истории переписки — после передачи его стоит отозвать (Settings → Developer settings → Tokens → Delete).

Ссылку разбери на `<owner>/<repo>`; проверь, что это `https://github.com/...`.

---

## Фаза 1. Что заливаем

Спроси одним вопросом:

```text
Что заливаем?
1. React-база с каталогом (workspace/product/code/) — то, что нужно разработчику (рекомендую)
2. Прототип (<путь, если есть>)
3. Весь проект — агент, контекст продукта, исследования, задачи, ДС и код
```

- **Весь проект** включает `workspace/product/context/` и материалы задач (интервью, инсайты). Предупреди об этом и уточни, что репозиторий приватный.
- Путь к прототипу неизвестен — спроси, не угадывай.

Подготовка:
1. **`.gitignore`** в корне заливаемой части. Нет — создай; есть — допиши недостающее (существующие строки не удаляй):
   ```gitignore
   node_modules/
   dist/
   build/
   storybook-static/
   .tmp/
   .env
   .env.*
   !.env.example
   .DS_Store
   *.log
   ```
   Для «всего проекта» корневой `.gitignore` уже есть — дописываются только недостающие строки.
2. **Проверка на секреты** до коммита: файлы `.env*`, `*.pem`, `*.key`, `credentials*`, `token*` и строки вида `ghp_`, `github_pat_`, `sk-`, `AKIA`, `-----BEGIN` в отслеживаемых файлах. Находка → показать пользователю, исключить, только потом коммит.
3. **Состав:** число файлов и объём (`git ls-files` / `git status --short` для подготовленного индекса), крупные файлы > 10 МБ — показать отдельно.
4. **Подтверждение:** «Заливаю <N файлов, X МБ> из <часть> в <ссылка>, ветка main. Подтверждаешь?»

---

## Фаза 2. Пуш одной операцией

### Вариант А. Часть проекта (React-база / прототип)

Отдельная рабочая копия в scratchpad, чтобы не трогать git продукта:

```bash
SRC="d:/…/workspace/product/code"           # заливаемая часть
WORK="<scratch>/handoff/<repo>"
mkdir -p "$WORK" && cd "$WORK"
git init -b main
# копия без служебного (robocopy на Windows / rsync)
robocopy "$SRC" . /E /XD node_modules dist build storybook-static .tmp .git /XF .env .env.* .DS_Store *.log
git add -A
git status --short | head            # контроль: нет node_modules, .env
git commit -m "<Продукт>: React-база и каталог Storybook"
git remote add origin https://github.com/<owner>/<repo>.git
git push -u origin main
```

`robocopy` возвращает коды 0–7 как успех — не считай их ошибкой.

### Вариант Б. Весь проект

Рабочее дерево продукта должно быть чистым (`git status`). Есть незакоммиченное — покажи список и спроси, коммитить ли это одним коммитом (только файлы продукта и `.gitignore`).

```bash
git remote add handoff https://github.com/<owner>/<repo>.git   # origin и template не трогаем
git push handoff main
```

### Авторизация

- Менеджер учётных данных / `gh` — обычный `git push`; при первом пуше откроется окно входа GitHub — предупреди пользователя заранее.
- Токен — только в одной команде, через заголовок, без записи в config и без URL с токеном:
  ```bash
  git -c credential.helper= \
      -c "http.https://github.com/.extraheader=AUTHORIZATION: basic $(printf 'x-access-token:%s' "$GH_TOKEN" | base64 -w0)" \
      push -u origin main
  ```
  `GH_TOKEN` задаётся только в окружении этой команды. После пуша проверь `git config --list --show-origin | grep -i -E "token|extraheader"` — пусто.

### Ошибки

| Ошибка | Что делать |
|---|---|
| `rejected (fetch first)` — в репозитории есть README / license | `git pull --rebase origin main`, затем `push`. Без `--force` |
| `403` / `Permission denied` | Нет прав у токена (нужен `repo`) или не тот аккаунт в менеджере учётных данных |
| `Repository not found` | Опечатка в ссылке или нет доступа к приватному репозиторию |
| Файл > 100 МБ | Исключить из заливки; предложить Git LFS отдельно |

---

## Фаза 3. README и запись

1. **README для разработчика** в корне заливаемой части (`code/README.md`). Есть — не перезаписывать, предложить дополнение. Содержимое по шаблону из [output-template.md](output-template.md): что это, состав (база, токены, каталог), как запустить (`npm install`, `npm run dev`, `npm run storybook`), откуда токены (Figma + `ds/foundation.md`), ссылка на Figma-файл из `ds/CONTRACT.md`.
   Для «всего проекта» корневой `README.md` — файл шаблона агента и не меняется; README разработчика — `code/README.md`, ссылку на него дай в финале.
2. **`product/project.md`** — дописать раздел «Репозиторий разработки» (шаблон в [output-template.md](output-template.md)): ссылка, ветка, что залито, дата. Существующее содержимое не трогать.
3. Закоммитить и запушить тем же способом: README — в рабочую копию (вариант А) или в репозиторий продукта (вариант Б); `project.md` — в репозиторий продукта коммитом только этого файла (вариант А: в `origin` продукта не пушить без указания пользователя — достаточно локального коммита и сообщения об этом).

---

## Фаза 4. Проверка и ссылка

1. `git ls-remote https://github.com/<owner>/<repo>.git` — есть `refs/heads/main`.
2. Список файлов в удалённом `main` (`git ls-tree -r --name-only origin/main` в рабочей копии) — нет `node_modules`, `.env`, `dist`.
3. Попроси пользователя открыть страницу репозитория: README отображается, ветка `main`.
4. Self-check из [checklist.md](checklist.md).
5. Финальный отчёт с HTTPS-ссылкой. Если использовался токен — напомни отозвать или оставить до следующей передачи осознанно.

---

## Фаза 5. Повторный запуск = обновление

1. Репозиторий — из `product/project.md`. Новый не создаём.
2. Спроси: правки в основную линию (`main`) или незаконченная итерация (отдельная ветка, например `design/<YYYY-MM-DD>-<тема>`)?
3. Вариант А: `git clone https://github.com/<owner>/<repo>.git <scratch>/handoff/<repo>` (или обновить существующую копию `git pull`), синхронизировать файлы из источника (`robocopy /MIR` с теми же исключениями, `.git` исключить), при отдельной ветке — `git switch -c <ветка>`; один коммит, один пуш.
   Вариант Б: `git push handoff main` или `git push handoff main:<ветка>`.
4. Обнови раздел в `project.md` (дата, последняя ветка, что обновлено).
5. В финале — что обновлено, в какую ветку; для ветки — ссылка на создание Pull Request (`https://github.com/<owner>/<repo>/compare/main...<ветка>`).
