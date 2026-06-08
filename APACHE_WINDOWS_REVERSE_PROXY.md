# Розгортання WUI за Apache на Windows

WUI слухає локальний TCP-порт (за замовчуванням `127.0.0.1:8080`) і обслуговує веб-інтерфейс
разом із WebSocket-ендпоінтом. Цей посібник ставить перед ним Apache 2.4 (на Windows)
як reverse proxy у двох варіантах:

- **Варіант A — HTTP**, доступний за адресою `http://<host>/` (або за підшляхом `/wui/`).
- **Варіант B — HTTPS із власним доменом**, доступний за адресою `https://wui.example.com/`.

> Тримайте WUI прив'язаним до `127.0.0.1` (config.yaml `server.host: "127.0.0.1"`), щоб
> сервіс був доступний **лише** через Apache, ніколи безпосередньо з мережі.

---

## Що очікує збірка (прочитайте це першим)

Вбудований Angular-застосунок припускає, що він розміщений у **корені домену**. Він постачається
з `<base href="/">` і викликає кілька ендпоінтів за **абсолютними** шляхами:

| Path | Призначення | Обов'язково? |
|------|---------|-----------|
| `/ws` | WebSocket — усі живі дані | **Так** (без нього застосунок марний) |
| `/api/v1/support` | Контакти підтримки у верхній панелі | Якщо ви задали `support.*` у конфізі |
| `/api/v1/health` | Перевірка стану | Опціонально |
| `/files`, `/ProxyHandler` | Web worker цифрового підпису | Лише якщо ви використовуєте підписання |
| `/config.json` | Перемикач "режиму підтримки" лише для relay | **Ні** — 404 очікуваний, застосунок переходить у локальний режим |

Практичний наслідок: **обслуговування WUI в корені домену — це чистий шлях**
(один `ProxyPass /` покриває все). Обслуговування за підшляхом на кшталт `/wui/`
працює, але кожен абсолютний шлях вище має проксіюватися в корені *окремо*, а
`<base href>` має бути переписаний. За можливості віддавайте перевагу виділеному
хосту/субдомену (Варіант B).

---

## Передумови

1. **Apache 2.4 для Windows** (наприклад, [Apache Lounge](https://www.apachelounge.com/),
   або Apache, що постачається з XAMPP). Цей посібник припускає `C:\Apache24`; підлаштуйте
   шляхи під вашу інсталяцію.

2. **Увімкніть необхідні модулі.** У `C:\Apache24\conf\httpd.conf` переконайтеся, що
   ці рядки присутні та **розкоментовані** (приберіть провідний `#`):

   ```apache
   LoadModule proxy_module          modules/mod_proxy.so
   LoadModule proxy_http_module     modules/mod_proxy_http.so
   LoadModule proxy_wstunnel_module modules/mod_proxy_wstunnel.so
   LoadModule rewrite_module        modules/mod_rewrite.so
   LoadModule headers_module        modules/mod_headers.so
   LoadModule substitute_module     modules/mod_substitute.so
   LoadModule filter_module         modules/mod_filter.so
   # Для Варіанта B (HTTPS) також:
   LoadModule ssl_module            modules/mod_ssl.so
   LoadModule socache_shmcb_module  modules/mod_socache_shmcb.so
   ```

3. **WUI запущений** (інтерактивно або як служба Windows — див.
   [`WINDOWS_SERVICE.md`](WINDOWS_SERVICE.md)) і відповідає на
   `http://127.0.0.1:8080/`.

Розмістіть блоки virtual-host нижче в кінці `httpd.conf` або в окремому
файлі (наприклад, `C:\Apache24\conf\extra\wui.conf`), який ви підключаєте через
`Include conf/extra/wui.conf`.

Після будь-якої зміни перевірте та перезапустіть:

```bat
C:\Apache24\bin\httpd.exe -t
C:\Apache24\bin\httpd.exe -k restart
```

---

## Варіант A — HTTP

### A1. У корені домену (рекомендовано для HTTP)

Якщо цей хост Apache виділено під WUI, обслуговуйте його в `/`. Усе просто працює:

```apache
<VirtualHost *:80>
    ServerName wui.local            ;# or the machine's hostname / IP

    ProxyPreserveHost On

    # WebSocket upgrade — must come before the catch-all ProxyPass
    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} websocket  [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/(.*)$  "ws://127.0.0.1:8080/$1"  [P,L]

    ProxyPass        /  http://127.0.0.1:8080/
    ProxyPassReverse /  http://127.0.0.1:8080/

    ErrorLog  "logs/wui-error.log"
    CustomLog "logs/wui-access.log" common
</VirtualHost>
```

Відкрийте `http://wui.local/` (або `http://<machine-ip>/`).

### A2. За підшляхом `/wui/` (коли корінь використовується чимось іншим)

Це також займає `/ws`, `/api/v1/`, `/files`, `/ProxyHandler` у вершині (див.
таблицю вище) і переписує base href SPA:

```apache
<VirtualHost *:80>
    ServerName cloud.example.com

    ProxyPreserveHost On

    # --- WebSocket: the UI connects to an ABSOLUTE /ws, so it lives at root ---
    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} websocket  [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/ws$  "ws://127.0.0.1:8080/ws"  [P,L]

    # --- Other absolute-root endpoints the bundle calls ---
    ProxyPass        /api/v1/       http://127.0.0.1:8080/api/v1/
    ProxyPassReverse /api/v1/       http://127.0.0.1:8080/api/v1/
    ProxyPass        /files         http://127.0.0.1:8080/files
    ProxyPassReverse /files         http://127.0.0.1:8080/files
    ProxyPass        /ProxyHandler  http://127.0.0.1:8080/ProxyHandler
    ProxyPassReverse /ProxyHandler  http://127.0.0.1:8080/ProxyHandler

    # --- The app itself under /wui/ — strip the prefix; backend serves at root ---
    ProxyPass        /wui/  http://127.0.0.1:8080/
    ProxyPassReverse /wui/  http://127.0.0.1:8080/
    RedirectMatch 301 ^/wui$ /wui/

    # Rewrite the SPA base href so relative assets resolve under /wui/.
    # SUBSTITUTE needs uncompressed HTML, so disable upstream compression.
    RequestHeader unset Accept-Encoding
    AddOutputFilterByType SUBSTITUTE text/html
    Substitute "s|<base href=\"/\">|<base href=\"/wui/\">|q"

    ErrorLog  "logs/wui-error.log"
    CustomLog "logs/wui-access.log" common
</VirtualHost>
```

Відкрийте `http://cloud.example.com/wui/`.

> `/config.json` усе одно поверне 404 — це правильно і нешкідливо (це має значення лише
> для режиму підтримки relay). Попередження в консолі `[support-config] falling back to local
> mode` є очікуваним.

---

## Варіант B — HTTPS із власним доменом

Рекомендовано для будь-якого розгортання, доступного з інтернету. Виділений хост (або субдомен
на кшталт `wui.example.com`) дозволяє одному `ProxyPass /` покрити кожен ендпоінт —
без поправок по кожному шляху, без переписування base-href.

### B1. Отримайте сертифікат

Або:

- **Справжній CA** — випустіть сертифікат для `wui.example.com` (наприклад,
  [win-acme](https://www.win-acme.com/) для Let's Encrypt на Windows), потім вкажіть
  у директивах нижче на випущені файли `.crt`/`.key` (або `fullchain`/`privkey`).
  З win-acme ви можете дозволити йому записати сертифікат і перезавантажити Apache за вас.
- **Самопідписаний** (LAN / тестування) — згенеруйте його за допомогою OpenSSL (постачається з
  Apache Lounge у `C:\Apache24\bin`):

  ```bat
  C:\Apache24\bin\openssl.exe req -x509 -nodes -days 825 -newkey rsa:2048 ^
    -keyout C:\Apache24\conf\ssl\wui.key ^
    -out    C:\Apache24\conf\ssl\wui.crt ^
    -subj   "/CN=wui.example.com"
  ```

### B2. Virtual hosts

Переконайтеся, що присутній `Listen 443` (він присутній, щойно завантажено `mod_ssl` через
стандартний `conf/extra/httpd-ssl.conf`, або додайте `Listen 443` самостійно).

```apache
# Redirect plain HTTP to HTTPS
<VirtualHost *:80>
    ServerName wui.example.com
    Redirect permanent / https://wui.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName wui.example.com

    SSLEngine on
    SSLCertificateFile      "C:/Apache24/conf/ssl/wui.crt"
    SSLCertificateKeyFile   "C:/Apache24/conf/ssl/wui.key"
    # If your CA gives a separate chain file:
    # SSLCertificateChainFile "C:/Apache24/conf/ssl/chain.crt"

    ProxyPreserveHost On
    # Tell the backend the original request was HTTPS
    RequestHeader set X-Forwarded-Proto "https"

    # WebSocket upgrade — must come before the catch-all ProxyPass.
    # wss:// is terminated here; the hop to the backend is plain ws:// on loopback.
    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} websocket  [NC]
    RewriteCond %{HTTP:Connection} upgrade [NC]
    RewriteRule ^/(.*)$  "ws://127.0.0.1:8080/$1"  [P,L]

    ProxyPass        /  http://127.0.0.1:8080/
    ProxyPassReverse /  http://127.0.0.1:8080/

    ErrorLog  "logs/wui-ssl-error.log"
    CustomLog "logs/wui-ssl-access.log" common
</VirtualHost>
```

Відкрийте `https://wui.example.com/`.

> **Підшлях через HTTPS?** Якщо ви маєте обслуговувати `https://example.com/wui/` замість
> виділеного хоста, поєднайте SSL-блок вище з проксі по кожному шляху та
> base-href `Substitute` з **Варіанта A2** — застосовуються ті самі компроміси.

---

## Перевірка

1. `http(s)://<host>/` (або `/wui/`) завантажує UI.
2. **DevTools браузера → Network → WS**: з'єднання `/ws` показує статус
   **101 Switching Protocols** і залишається відкритим. Якщо показує 200/404/400, переписування
   WebSocket не спрацьовує — перевірте завантаження модулів і порядок правил.
3. Блок підтримки у верхній панелі з'являється, коли `support.viber/telegram/phone` задані
   в `config.yaml` *і* `/api/v1/support` доступний через Apache.

---

## Усунення несправностей

| Симптом | Причина / виправлення |
|---------|-------------|
| UI завантажується, але немає живих даних; WS не підключається | `mod_proxy_wstunnel` не завантажено, або WebSocket-`RewriteRule` стоїть **після** `ProxyPass /`. Завантажте модуль; розмістіть WS-правила першими. |
| `[support-config] falling back to local mode` (config.json 404) | **Очікувано.** `/config.json` лише для relay; ігноруйте. |
| Відсутні контакти підтримки | `/api/v1/support` не проксіюється (конфігурації з підшляхом), або `support.*` порожній у `config.yaml`. |
| Порожня сторінка / 404 на ресурсах під `/wui/` | base-href `Substitute` не застосовано — переконайтеся, що `mod_substitute` + `mod_filter` завантажені і присутній `RequestHeader unset Accept-Encoding` (gzip ламає SUBSTITUTE). |
| `AH00526 ... SSLCertificateFile` під час запуску | Неправильний шлях до сертифіката або зворотні слеші. Використовуйте прямі слеші у шляхах. |
| Цифрове підписання не вдається | `/files` і `/ProxyHandler` не проксіюються (лише підшлях). |
| Apache не запускається, порт зайнятий | Інша служба тримає 80/443 (IIS, Skype). Зупиніть її або змініть `Listen`. |

Корисні команди:

```bat
C:\Apache24\bin\httpd.exe -t          ;# validate config
C:\Apache24\bin\httpd.exe -M          ;# list loaded modules
C:\Apache24\bin\httpd.exe -k restart  ;# restart
```

Логи: `C:\Apache24\logs\` (`wui-error.log`, `wui-ssl-error.log`).
