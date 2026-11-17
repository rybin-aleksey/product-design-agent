# Шаблоны

Имена эндпоинтов, переменных и полей — сверять с README бандла.

## План теста — `tasks/<TASK-ID>/artifacts/validate/ut-plan.md`

```markdown
# План юзер-теста

**Дата:** <YYYY-MM-DD>
**Прототип:** <живая ссылка> (<сервис / свой сервер>)
**Путь подключения:** <1 — http / 2 — https через ngrok>

## Сценарии

| id | Название | Воронка | Цель | Первый клик | Источник |
|---|---|---|---|---|---|
| server-create | Создание сервера | /dashboard → /servers/new → /servers | /servers | /dashboard: `create-server` | registry + заметка Dashboard «Поведение» |

## Гипотезы

| id | Формулировка | Сценарий | Метрика | Порог | Шаг | Источник |
|---|---|---|---|---|---|---|
| H1 | Пользователь находит создание сервера без подсказок | server-create | firstclick | ≥ 0.7 | — | task.md, инсайт #3 |
| H2 | Большинство доходит до созданного сервера | server-create | completion | ≥ 0.8 | — | PRD, раздел «Цели» |

Пороги утверждены пользователем: <да / дата>.

## Разметка data-track
- `create-server` — кнопка «Создать сервер» (Dashboard)
- `error-topup` — ошибка пополнения (Balance)

## Ссылки U-теста
- server-create: <prototypeUrl>?ut_task=server-create

## Раунды
| Раунд | Дата | Респондентов | Ключевые метрики | Правки после |
|---|---|---|---|---|
```

## `hypotheses.json`

```json
[
  {
    "id": "H1",
    "text": "Пользователь находит создание сервера без подсказок",
    "flow": "server-create",
    "measure": "firstclick",
    "target": { "op": ">=", "value": 0.7 }
  },
  {
    "id": "H3",
    "text": "На шаге выбора тарифа отваливается не больше 20%",
    "flow": "server-create",
    "measure": "dropoff",
    "step": "/servers/new",
    "target": { "op": "<=", "value": 0.2 }
  }
]
```

---

## План изменений на сервере

```text
План на сервере <IP>:
- Node: <версия есть / установлю <версию>>
- Сборщик: /opt/ut-dashboard/server, данные — /var/lib/ut-dashboard
- Служба ut-collector на порту <порт> (занятые сейчас: <список>, VPN: <порт> — не трогаю)
- <Путь 1:> порт <порт> наружу — <ufw активен: добавлю правило / ufw выключен: не трогаю>; проверь панель провайдера
- <Путь 2:> ngrok + служба ut-tunnel → https://<домен>; токен ngrok останется только в конфиге ngrok на сервере
- Защита чтения метрик (DASH_KEY): <рекомендую включить / не включаем>
Ок?
```

## systemd: сборщик

`/etc/systemd/system/ut-collector.service`:

```ini
[Unit]
Description=UT dashboard collector
After=network.target

[Service]
WorkingDirectory=/opt/ut-dashboard/server
Environment=PORT=<порт>
Environment=DATA_DIR=/var/lib/ut-dashboard
EnvironmentFile=-/etc/ut-dashboard.env
ExecStart=/usr/bin/node server.js
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

`/etc/ut-dashboard.env` (права 600, только если включён ключ):

```text
DASH_KEY=<секрет>
```

## systemd: туннель (путь 2)

`/etc/systemd/system/ut-tunnel.service`:

```ini
[Unit]
Description=ngrok tunnel for UT collector
After=network-online.target ut-collector.service
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/ngrok http <порт> --url=https://<домен> --log=stdout
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

(Путь к бинарю — по факту установки: `command -v ngrok`.)

## Шаги ngrok для пользователя (путь 2)

```text
1. Открой ngrok.com → Sign up. Если GitHub привязан к mail.ru / yandex и т.п. — регистрируйся через Google.
2. В кабинете: Getting Started → Your Authtoken → скопируй токен.
3. Universal Gateway → Domains → возьми бесплатный постоянный домен (вида xxx.ngrok-free.app).
4. Пришли мне токен и домен. Токен я пропишу только в ngrok на сервере, в проект он не попадёт.
```

---

## Трекер в `index.html`

```html
  <script src="/track.js" data-endpoint="<АДРЕС СБОРА>/collect"></script>
</body>
```

## `dashboard/app/config.js`

```js
window.DASHBOARD_CONFIG = {
  endpoint: "<АДРЕС СБОРА>",                 // путь 1: http://<IP>:<порт> · путь 2: https://<домен>
  prototypeUrl: "<живая ссылка>/<стартовый экран>", // не «/» с дев-индексом
  screens: { "/dashboard": "Dashboard", "/servers/new": "Создание сервера" },
  flows: [
    {
      id: "server-create",
      name: "Создание сервера",
      funnel: ["/dashboard", "/servers/new", "/servers"],
      goal: "/servers",
      firstClick: { screen: "/dashboard", target: "create-server" }
    }
  ],
  key: ""                                     // DASH_KEY или пусто; файл не коммитится
};
```

---

## Раздел в `workspace/product/project.md`

```markdown
## Юзер-тесты

- **Сбор:** <АДРЕС СБОРА> (служба ut-collector на <IP>, данные — /var/lib/ut-dashboard)
- **Путь:** <1 — http / 2 — ngrok https://<домен>, служба ut-tunnel>
- **Дашборд:** <локально: workspace/product/dashboard/app / http://<IP>:<порт>>
- **Защита чтения:** <DASH_KEY включён / нет> (ключ здесь не хранится)
- **План и гипотезы:** tasks/<TASK-ID>/artifacts/validate/ut-plan.md
- **Последнее обновление:** <YYYY-MM-DD> — <что>
```

---

## Итог

```text
Дашборд юзер-тестов подключён.
- Путь: <свой сервер (http) | Vercel + сбор на сервере через ngrok (https)>
- Сбор: <АДРЕС СБОРА> (health ✓), службы: ut-collector<, ut-tunnel>; данные на сервере
- Трекер встроен в прототип (data-endpoint → сбор), разметка data-track: <N> элементов
- Сценарии: <N> · гипотезы: <N, пороги утверждены / пропущено>
- Дашборд: <как открыть>
- План теста: tasks/<TASK-ID>/artifacts/validate/ut-plan.md
- Ссылки U-теста: «Сценарии» → «Ссылка для теста» (prototypeUrl?ut_task=<id>)

Прогон: раздай ссылки U-теста (предупреди респондентов, что поведение записывается) → «Обновить» → «Зафиксировать раунд» → «Находки → задание» агенту → правки → передеплой → ретест.
<Если пароль / токены были в чате:> Смени пароль сервера; ngrok-токен при необходимости перевыпусти в кабинете.
```
