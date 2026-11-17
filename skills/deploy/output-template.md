# Шаблоны

## Выбор пути (фаза 0)

```text
Куда публикуем прототип?
1. Сервис — Vercel (рекомендую) / Netlify / Cloudflare Pages: HTTPS-ссылка, обновление по пушу в GitHub
2. Свой сервер (VPS) — ссылка вида http://<IP>:<порт>, без домена и HTTPS
```

---

## Конфиги SPA-fallback

`workspace/product/code/vercel.json`:

```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

`workspace/product/code/public/_redirects` (Netlify, Cloudflare Pages):

```text
/*    /index.html   200
```

nginx — `/etc/nginx/sites-available/<app>`:

```nginx
server {
    listen <порт>;
    listen [::]:<порт>;
    server_name _;

    root /var/www/<app>;
    index index.html;

    location / {
        try_files $uri /index.html;
    }

    location /assets/ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

```bash
ln -sf /etc/nginx/sites-available/<app> /etc/nginx/sites-enabled/<app>
nginx -t && systemctl reload nginx
```

---

## Браузерные шаги: сервис (A2)

Vercel:

```text
1. Открой vercel.com → Sign Up / Log In → Continue with GitHub.
2. Add New… → Project.
3. Найди репозиторий <repo> → Import (если его нет в списке — Adjust GitHub App Permissions и дай доступ к репозиторию).
4. Framework Preset — Vite (определится сам). Build Command — npm run build, Output Directory — dist.
   <Если в репозитории весь проект:> Root Directory → Edit → workspace/product/code.
5. Deploy. Дождись «Congratulations» и пришли мне ссылку на проект.
```

Netlify:

```text
1. app.netlify.com → Add new site → Import an existing project → GitHub.
2. Выбери <repo>. Build command — npm run build, Publish directory — dist
   <Base directory — workspace/product/code, если в репозитории весь проект>.
3. Deploy site → пришли ссылку вида https://<site>.netlify.app.
```

Cloudflare Pages:

```text
1. dash.cloudflare.com → Workers & Pages → Create application → Pages → Connect to Git.
2. Выбери <repo>. Framework preset — Vite, Build command — npm run build, Output — dist
   <Root directory — workspace/product/code, если в репозитории весь проект>.
3. Save and Deploy → пришли ссылку вида https://<project>.pages.dev.
```

Vercel — снять защиту (A3):

```text
Если ссылка в приватном окне просит войти в Vercel:
Проект → Settings → Deployment Protection → Vercel Authentication → выключить → Save.
Бери ссылку из поля Domains на странице проекта (<project>.vercel.app), а не адрес конкретного деплоя.
```

---

## Свой сервер (B1)

Запрос доступа:

```text
Для публикации на твой сервер нужны:
1. IP-адрес сервера
2. Пользователь (обычно root)
3. Вход: лучше по SSH-ключу — тогда пароль не придётся присылать в чат.
```

Добавить ключ (пользователь выполняет сам в PowerShell и вводит пароль сервера):

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh root@<IP> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

План перед изменениями:

```text
План на сервере <IP> (<ОС>):
- Занятые порты сейчас: <список> (VPN: <порт> — не трогаю)
- nginx: <уже есть / установлю>
- Порт прототипа: <порт>
- Файлы: /var/www/<app> (сборка), /etc/nginx/sites-available/<app> (новый конфиг; существующие не меняю)
- Firewall: <ufw активен — добавлю правило <порт>/tcp / ufw выключен — не трогаю>; проверь порт в панели провайдера
Ок?
```

---

## Раздел в `workspace/product/project.md`

```markdown
## Публикация

- **Ссылка:** <https://<project>.vercel.app | http://<IP>:<порт>>
- **Путь:** <сервис: Vercel / Netlify / Cloudflare Pages | свой сервер>
- **Источник:** <репозиторий <ссылка>, ветка main, Root Directory <…> | сборка dist, /var/www/<app>, nginx на порту <порт>>
- **SPA-fallback:** <vercel.json / public/_redirects / nginx try_files>
- **Последнее обновление:** <YYYY-MM-DD> — <что>
```

Пароли, токены и ключи сюда не пишутся.

---

## Итог

```text
Продукт опубликован.
- Путь: <сервис (Vercel / Netlify / Cloudflare) | свой сервер по IP>
- Живая ссылка: <URL | http://IP:порт>
- SPA-маршрутизация: внутренние экраны открываются по прямой ссылке и после обновления страницы
- Записано: workspace/product/project.md, раздел «Публикация»
<Свой сервер:> Адрес по http — браузер покажет «не защищено», для теста это нормально. Домен и HTTPS — отдельная задача.
<Если пароль / токен были в чате:> Смени пароль сервера / удали токен — они больше не нужны.

Сохрани ссылку — по ней пойдут юзер-тест и метрики.
Обновить публикацию: запусти deploy снова<; для сервиса достаточно пуша в main>.
```
