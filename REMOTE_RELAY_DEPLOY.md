# Relay-сервер WUI — Посібник із розгортання

Цільова платформа: Ubuntu 22.04 / 24.04 LTS.
Приклад домену протягом усього тексту: `web.che.ck.ua`.

## 1. Передумови сервера

```bash
sudo apt update
sudo apt install -y nginx certbot python3-certbot-nginx
```

Спрямуйте `A`-запис домену на сервер перед продовженням, щоб `certbot` міг
завершити виклик HTTP-01.

## 2. Створіть файл auth-токена

> **Примітка:** Спільний bearer-токен поступово виводиться з ужитку. Агенти WUI тепер підключаються до
> relay через **відкритий ендпоінт** і авторизують доступ підтримки через
> виданий relay PIN + SessionID — див.
> [`REMOTE_RELAY_AUTH.md`](REMOTE_RELAY_AUTH.md) щодо засобів контролю, які relay має
> реалізувати в цій моделі. Поки що збережіть токен нижче, якщо ваша збірка relay
> усе ще його вимагає; надалі розглядайте його як опціональний/лише для міграції. Коли
> `WUI_RELAY_AUTH_TOKEN` порожній, relay запускає ендпоінт агента у **відкритому
> режимі** і логує `agent_auth=open (tokenless)` під час запуску. Два регулятори зловживань по IP
> охороняють цей ендпоінт:
> `WUI_RELAY_AGENT_RATE_PER_MIN` (нові з'єднання/хв/IP, за замовчуванням 60) і
> `WUI_RELAY_MAX_AGENT_CONNS_PER_IP` (одночасні тунелі/IP, за замовчуванням 0 = вимкнено,
> тож парки за NAT-агрегацією не дроселюються).

Relay вимагає спільного bearer-токена для агентів. Згенеруйте його та помістіть
у env-файл, читабельний для systemd (скрипт інсталяції відмовляється продовжувати без нього):

```bash
sudo install -d -m 750 /etc/wui-relay
sudo tee /etc/wui-relay/relay.env >/dev/null <<ENV
WUI_RELAY_AUTH_TOKEN=$(openssl rand -hex 32)
ENV
sudo chmod 640 /etc/wui-relay/relay.env
```

Скрипт інсталяції виконає `chown` цього файлу на `root:wui-relay` після створення
системного користувача. Скопіюйте той самий токен у конфіг кожного локального екземпляра WUI:

```yaml
# configs/config.yaml on each customer WUI box
remote:
  enabled: true
  server_url: "wss://web.che.ck.ua/api/v1/agent/ws"
  auth:
    enabled: true
    token_secret: "<same token>"
```

## 3. Встановіть relay

Завантажте останній tarball релізу з релізу `relay-latest` репозиторію:

```bash
cd /tmp
curl -L -o wui-relay.tar.gz \
    https://github.com/ruslanhut/wui/releases/download/relay-latest/wui-relay-linux-amd64.tar.gz
mkdir -p wui-relay && tar -xzf wui-relay.tar.gz -C wui-relay
cd wui-relay
sudo ./install.sh
```

Скрипт:

1. Створює системного користувача `wui-relay` і `/etc/wui-relay/`.
2. Перевіряє, що `relay.env` містить `WUI_RELAY_AUTH_TOKEN`.
3. Встановлює `/usr/local/bin/wui-relay`, systemd-юніт і (лише при першому встановленні)
   стандартний `/etc/wui-relay/config.yaml`.
4. Вмикає та запускає `wui-relay.service`.
5. Розміщує сайт nginx у `/etc/nginx/sites-available/wui-relay`, що вказує на
   `127.0.0.1:9000`.

Перевірте:

```bash
systemctl status wui-relay
curl -s http://127.0.0.1:9000/api/v1/health
journalctl -u wui-relay -f
```

## 4. Увімкніть HTTPS

Поставлений конфіг nginx використовує `server_name web.che.ck.ua;` — відредагуйте його, якщо ваш
домен відрізняється, потім:

```bash
sudo nginx -t
sudo systemctl reload nginx
sudo certbot --nginx -d web.che.ck.ua
```

Certbot автоматично додає server-блок `443` і перенаправлення HTTP→HTTPS та
планує таймер оновлення.

Перевірте публічний ендпоінт димовим тестом:

```bash
curl -s https://web.che.ck.ua/api/v1/health
```

## 5. Підключіть агента

На клієнтській машині WUI задайте `remote.server_url` як
`wss://web.che.ck.ua/api/v1/agent/ws` і `remote.auth.token_secret` як
токен із кроку 2. Увімкніть віддалений доступ з UI. У журналі relay ви
маєте побачити:

```
Agent registered agent_id=... hostname=... fiscal_numbers=N
```

## 6. Оновлення

Щоб оновити бінарник, завантажте новий tarball і повторно запустіть `sudo ./install.sh`.
Скрипт зупиняє службу, замінює бінарник і статичну збірку UI підтримки
у `/var/www/wui-relay/`, і знову запускає службу. Він не
перезаписуватиме `/etc/wui-relay/config.yaml`, `relay.env` чи наявний
`/etc/nginx/sites-available/wui-relay` (certbot редагує цей файл на місці, щоб
додати блок :443, тож збереження його при оновленні уникає стирання HTTPS).

### Оновлення до Phase 3b (браузерний UI підтримки)

Одноразове злиття конфігу nginx потрібне при переході з relay Phase 2
(чистий WebSocket-проксі на `/`) до Phase 3b (статична Angular-збірка на `/` з
API, проксійованим під `/api/v1/`). Вручну відредагуйте
`/etc/nginx/sites-available/wui-relay`:

1. Змініть єдиний проксі-блок `location /` так, щоб його `proxy_pass` був вкладений
   під `location /api/v1/` (перемістіть наявний блок).
2. Додайте на початку блоку `server { ... }`, над наявними рядками:
   ```nginx
   root  /var/www/wui-relay;
   index index.html;

   location = /config.json {
       add_header Cache-Control "no-store";
       try_files $uri =404;
   }
   ```
3. Додайте в кінці server-блоку:
   ```nginx
   location ~* \.(?:js|css|woff2?|ttf|svg|png|jpg|jpeg|gif|ico)$ {
       expires 30d;
       add_header Cache-Control "public, immutable";
       try_files $uri =404;
   }

   location / {
       try_files $uri $uri/ /index.html;
   }
   ```
4. `sudo nginx -t && sudo systemctl reload nginx`

Поставлений `deploy/nginx-wui-relay.conf` у tarball релізу є
повністю злитим еталоном, якщо ви хочете порівняти його з вашою поточною копією.
Збережіть будь-які рядки `listen 443 ssl;` / `ssl_certificate ...`, які certbot додав
до наявного файлу — вони залишаються в тому самому server-блоці.

### Автоматизовані оновлення через GitHub Actions

Workflow `release-relay.yml` має job `deploy`, який підключається по SSH до relay-
сервера як **root** після успішної збірки і запускає `install.sh` автоматично.
install.sh ідемпотентний: він зупиняє службу, замінює бінарник, повторно запускає
`systemctl daemon-reload` і знову запускає службу. Він не перезаписуватиме
`/etc/wui-relay/config.yaml` чи `relay.env`.

**Необхідні секрети репозиторію** (Settings → Secrets and variables → Actions):

| Secret | Значення |
|--------|-------|
| `RELAY_SSH_HOST` | `web.che.ck.ua` (або сирий IP) |
| `RELAY_SSH_USER` | `root` |
| `RELAY_SSH_KEY`  | Вміст приватного ключа. Відповідний публічний ключ уже має бути в `/root/.ssh/authorized_keys` на сервері. |
| `RELAY_SSH_PORT` | Опціонально. За замовчуванням 22. |

**Згенеруйте та встановіть deploy-ключ** (запустіть один раз на вашій робочій станції, не на
сервері):

```bash
ssh-keygen -t ed25519 -f ~/.ssh/wui-relay-deploy -C "wui-relay ci deploy" -N ""
# Copy the public key to the server:
ssh-copy-id -i ~/.ssh/wui-relay-deploy.pub root@web.che.ck.ua
# Put the contents of ~/.ssh/wui-relay-deploy (the PRIVATE key) into the
# RELAY_SSH_KEY GitHub secret.
```

Для глибокого захисту обмежте ключ лише командою інсталяції, відредагувавши
відповідний рядок у `/root/.ssh/authorized_keys`:

```
command="/tmp/wui-relay/install.sh",no-port-forwarding,no-agent-forwarding,no-X11-forwarding ssh-ed25519 AAAA...
```

(Пропустіть, якщо хочете тримати ключ придатним для звичайних root-шелів.)

**Перший deploy усе ще має бути ручним:** `install.sh` відмовляється запускатися, доки
`/etc/wui-relay/relay.env` не існує з `WUI_RELAY_AUTH_TOKEN`. Виконайте bootstrap один раз
через розділи 2–4 вище, потім кожен наступний push у `master`, що торкається
коду relay, буде розгортатися автоматично.

## 7. Видалення

```bash
sudo systemctl disable --now wui-relay
sudo rm /etc/systemd/system/wui-relay.service /usr/local/bin/wui-relay
sudo rm -rf /etc/wui-relay
sudo userdel wui-relay
sudo rm /etc/nginx/sites-enabled/wui-relay /etc/nginx/sites-available/wui-relay
sudo systemctl reload nginx
```
