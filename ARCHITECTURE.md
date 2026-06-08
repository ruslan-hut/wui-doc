# Архітектура WUI

## Огляд системи

```
                        +--------------------------------------------------+
                        |              WUI Application Binary               |
                        |                                                  |
                        |   +------------------------------------------+   |
                        |   |          Embedded Angular SPA            |   |
                        |   |        (internal/app/static/)            |   |
                        |   +------------------------------------------+   |
                        |        |                              ^          |
                        |        | HTTP GET /*                  |          |
                        |        v                              |          |
                        |   +------------------------------------------+   |
                        |   |         Chi HTTP Router (server.go)      |   |
                        |   |                                          |   |
                        |   |  GET /ws ......... WebSocket Upgrade     |   |
                        |   |  GET /health ..... Health Check          |   |
                        |   |  GET /* .......... Static Files (SPA)    |   |
                        |   +------------------------------------------+   |
                        |        |                                         |
                        |        | Upgrade to ws://                        |
                        |        v                                         |
                        |   +------------------------------------------+   |
                        |   |          WebSocket Layer                  |   |
                        |   |                                          |   |
                        |   |  Hub ---- Client Registry & Broadcast    |   |
                        |   |   |                                      |   |
                        |   |  Client -- ReadPump / WritePump          |   |
                        |   |   |         Ping/Pong Keepalive          |   |
                        |   |   |                                      |   |
                        |   |  MessageHandler -- Routes by type        |   |
                        |   |   |-- QueryHandler (device.query.*)      |   |
                        |   |   |-- CommandHandler (reload/reports)     |   |
                        |   |   |-- SettingsHandler (update-setting)    |   |
                        |   |   +-- LicenseHandler (license.command)    |   |
                        |   +------------------------------------------+   |
                        |        |                                         |
                        |        v                                         |
                        |   +------------------------------------------+   |
                        |   |          Service Layer                    |   |
                        |   |                                          |   |
                        |   |  DatabaseReader ... POS DB queries       |   |
                        |   |  INILoader ........ settings.ini R/W     |   |
                        |   |  OLEService ....... Fiscal device ops    |   |
                        |   |  ReloadService .... Full data refresh    |   |
                        |   |  SettingsService .. Config management    |   |
                        |   |  LicenseService ... License ordering     |   |
                        |   +------------------------------------------+   |
                        |        |                    |                    |
                        |        v                    v                    |
                        |   +----------------+  +------------------+       |
                        |   | POS Databases  |  | OLE/COM Bridge   |       |
                        |   | (SQLite files) |  | (VBScript/COM)   |       |
                        |   +----------------+  +------------------+       |
                        |        |                    |                    |
                        +--------|--------------------|---------+----------+
                                 |                    |         |
                                 v                    v         v
                        +----------------+  +--------+  +-------------+
                        | POS Device DBs |  | Fiscal |  | License     |
                        | (*.db files)   |  | Device |  | Server      |
                        +----------------+  +--------+  | (HTTP API)  |
                                                        +-------------+
```

---

## Архітектура backend (Go)

```
cmd/wui/
+-- main.go                         Точка входу, DI-звʼязування, коректне завершення роботи

internal/
|
+-- app/
|   +-- server.go                   Chi-роутер, embed статики, WS upgrade
|
+-- config/
|   +-- config.go                   Конфігурація YAML + env (cleanenv)
|
+-- domain/
|   +-- device/
|   |   +-- device.go               Структури Device, DeviceConfig, Settings
|   +-- view/
|       +-- device_view.go          Зручні для frontend view-моделі
|
+-- handler/
|   +-- websocket_handler.go        HTTP -> WebSocket upgrade (gorilla)
|   +-- api/
|       +-- devices_handler.go      Застарілий REST: список пристроїв
|       +-- reload_handler.go       Застарілий REST: запуск перезавантаження
|
+-- websocket/                      === Ядро WebSocket-комунікації ===
|   +-- hub.go                      Реєстр клієнтів, цикл broadcast
|   +-- client.go                   Горутини на зʼєднання (R/W pumps)
|   +-- conn_interface.go           Абстракція зʼєднання (тестованість)
|   +-- conn_wrapper.go             Обгортка реального зʼєднання
|   +-- message.go                  Конверт повідомлення, константи типів
|   +-- message_handler.go          Центральний роутер: тип -> обробник
|   +-- query_handler.go            device.query.list/settings/log
|   +-- query_handler_database.go   device.query.shifts/checks/operators
|   +-- command_handler.go          reload.trigger, звіти X/Z/Period
|   +-- settings_handler.go         device.command.update-setting
|   +-- license_handler.go          license.command.order
|   +-- testing.go                  Допоміжні засоби для тестів
|   +-- mocks_test.go              Mock-реалізації
|
+-- service/                        === Бізнес-логіка ===
|   +-- database_reader.go          Читання POS SQLite БД, кеш, broadcast
|   +-- ini_loader.go               Завантаження/збереження settings.ini (Win-1251)
|   +-- ole_service.go              Оркестрація OLE (звіти, ліцензія)
|   +-- ole_interface.go            Інтерфейс OLEClient
|   +-- ole_windows.go              Windows OLE через VBScript-міст
|   +-- ole_native_windows.go       Windows OLE через прямий COM
|   +-- ole_stub.go                 Заглушка для не-Windows
|   +-- ole_mock.go                 Mock OLE для тестів
|   +-- reload_service.go           Повне перезавантаження: INI + БД + ліцензії
|   +-- settings_service.go         Читання/запис налаштувань, читання логу частинами
|   +-- license_http.go             HTTP-клієнт сервера ліцензій
|   +-- license_order_service.go    Замовлення ліцензій + кешування
|   +-- websocket_adapters.go       Адаптери для уникнення циклів імпорту
|
+-- logger/
|   +-- logger.go                   Інтерфейс структурованого логування (slog)
|   +-- init.go                     Ініціалізація логера
|
+-- pkg/olexml/
    +-- olexml.go                   Парсер XML-відповідей OLE
```

### Потік даних backend

```
                    WebSocket Message (JSON)
                           |
                           v
                  +------------------+
                  | MessageHandler   |
                  | (routes by type) |
                  +--------+---------+
                           |
          +-------+--------+--------+--------+
          |       |        |        |        |
          v       v        v        v        v
       Query   Query    Command  Settings License
       Handler  DB Hndl  Handler  Handler  Handler
          |       |        |        |        |
          v       v        v        v        v
     +--------+ +------+ +------+ +------+ +------+
     | INI    | | DB   | | OLE  | | INI  | | HTTP |
     | Loader | | Readr| | Svc  | | Save | | Lic  |
     +--------+ +------+ +------+ +------+ +------+
          |       |        |                   |
          v       v        v                   v
     settings  POS DBs  Fiscal            License
       .ini   (SQLite)  Device             Server
```

### Ключові патерни backend

| Патерн | Реалізація | Призначення |
|---------|---------------|---------|
| **Adapter** | `websocket_adapters.go` | Розрив циклів імпорту між `websocket` і `service` |
| **Double Pointer** | `**device.Settings` | ReloadService + SettingsService спільно використовують актуальне посилання на налаштування |
| **Hub/Client** | `hub.go` + `client.go` | Конкурентне керування зʼєднаннями за допомогою горутин |
| **Progressive Load** | `database_reader.go` | Broadcast часткових даних у міру завантаження кожного пристрою |
| **Platform Build Tags** | `ole_windows.go` / `ole_stub.go` | OLE лише для Windows, заглушка на інших ОС |
| **Encoding Detection** | `utf8.Valid()` у `ini_loader.go` | Автовизначення Win-1251 чи UTF-8 |

---

## Архітектура frontend (Angular)

```
frontend/src/
|
+-- main.ts                          Bootstrap Angular-застосунку
+-- index.html                       Оболонка HTML + шрифт Material Symbols
+-- styles.scss                      Точка входу глобальних стилів
+-- styles/
|   +-- _variables.scss              CSS custom properties (кольори, відступи)
|   +-- _themes.scss                 Визначення світлої / темної теми
|   +-- _typography.scss             Шрифти Montserrat + Commissioner
|   +-- _buttons.scss                Варіанти кнопок
|   +-- _forms.scss                  Елементи форм
|   +-- _tables.scss                 Розкладка таблиць
|   +-- _layout.scss                 Утиліти розкладки
|   +-- _modals.scss                 Діалог / нижня панель (bottom sheet)
|   +-- _receipt.scss                Відображення чека
|   +-- _skeleton.scss               Skeleton-завантажувачі
|   +-- _states.scss                 Стилі на основі станів
|   +-- _animations.scss             Переходи
|
+-- app/
    +-- app.ts                       Кореневий компонент
    +-- app.config.ts                Провайдери, APP_INITIALIZER (WS + тема)
    +-- app.routes.ts                Визначення маршрутів (lazy-loaded)
    |
    +-- core/                        === Singleton-сервіси та моделі ===
    |   +-- services/
    |   |   +-- websocket.service.ts     Життєвий цикл WS, перепідключення, heartbeat
    |   |   +-- device.service.ts        Запити, команди та стан пристроїв
    |   |   +-- settings.service.ts      Керування налаштуваннями пристрою
    |   |   +-- theme.service.ts         Перемикання світлої/темної теми
    |   |   +-- notification.service.ts  Toast / сповіщення
    |   |   +-- sidebar.service.ts       Стан відкриття/закриття бічної панелі
    |   |   +-- register-context.service.ts  Контекст вибраної каси
    |   |   +-- receipt-pdf.service.ts   Генерація PDF для чеків
    |   |
    |   +-- models/
    |   |   +-- settings.model.ts    Інтерфейси Device, DeviceConfig, Settings
    |   |
    |   +-- utils/
    |       +-- date.util.ts         Форматування дат (локальний час, без UTC)
    |       +-- doc-type.util.ts     Класифікація типів документів
    |       +-- refresh-phase.util.ts  Керування станом оновлення даних
    |
    +-- layout/                      === Компоненти оболонки застосунку ===
    |   +-- shell/                   Головна розкладка (header + sidebar + outlet)
    |   +-- header/                  Верхня панель: заголовок, статус, перемикач теми
    |   +-- sidebar/                 Меню навігації: список кас + посилання на функції
    |   +-- register-context/        Віджет вибору каси
    |
    +-- features/                    === Сторінки функцій (lazy-loaded) ===
    |   +-- registers/               Список кас + статус
    |   +-- sales/                   Відображення даних продажів
    |   +-- payment-methods/         Способи оплати для каси
    |   +-- operators/               Список операторів
    |   +-- shifts/                  Список змін + фільтрація за датою
    |   +-- checks/                  Список чеків для зміни
    |   +-- offline-status/          Статус онлайн/офлайн
    |   +-- reports/                 Генерація звітів X / Z / Period
    |   +-- settings/                Редактор налаштувань пристрою
    |   +-- log-viewer/              Перегляд файлу логу (частинами)
    |   +-- licenses/                Інформація про ліцензії + замовлення
    |   +-- alerts/                  Центр сповіщень
    |
    +-- shared/                      === Перевикористовувані компоненти ===
        +-- components/
            +-- skeleton-loader.component.ts   Плейсхолдер завантаження
```

### Ієрархія компонентів frontend

```
AppComponent (app.ts)
  |
  +-- ShellComponent (layout/shell/)
       |
       +-- HeaderComponent (layout/header/)
       |    +-- RegisterContextComponent
       |    +-- Connection status indicator
       |    +-- Theme toggle button
       |
       +-- SidebarComponent (layout/sidebar/)
       |    +-- Register list
       |    +-- Feature navigation links
       |
       +-- <router-outlet>
            |
            +-- RegistersComponent
            +-- SalesComponent
            +-- PaymentMethodsComponent
            +-- OperatorsComponent
            +-- ShiftsComponent
            +-- ChecksComponent
            +-- OfflineStatusComponent
            +-- ReportsComponent
            +-- SettingsComponent
            +-- LogViewerComponent
            +-- LicensesComponent
            +-- AlertsComponent
```

### Взаємодія сервісів frontend

```
+--------------------------------------------------+
|              Angular Components                   |
|  (registers, shifts, checks, reports, ...)        |
+----------+---+---+---+---+-----------------------+
           |   |   |   |   |
           v   v   |   |   v
    +----------+ +-+---+-+ +---------------+
    | Device   | |Setting| | Notification  |
    | Service  | |Service| | Service       |
    +----+-----+ +---+---+ +---------------+
         |           |
         v           v
    +-------------------------------+
    |       WebSocket Service       |
    |                               |
    |  - query<T>() .... req/res    |
    |  - send() ........ fire msg   |
    |  - messages$ ..... broadcast   |
    |  - connected ..... signal     |
    |  - reconnect ..... auto       |
    |  - heartbeat ..... ping 30s   |
    +---------------+---------------+
                    |
                    | ws://host/ws
                    v
            +---------------+
            |  Go Backend   |
            +---------------+
```

### Керування станом frontend

```
WebSocket Service                  Device Service
  connected: Signal<boolean>         devices: Signal<Device[]>
  messages$: Subject<Message>        loading: Signal<boolean>
                                     dataLoaded: Map<string, boolean>
                                     licenseLoaded: Map<string, boolean>
Theme Service                      Sidebar Service
  theme: Signal<'light'|'dark'>      isOpen: Signal<boolean>

Register Context Service
  selectedRegister: Signal<Device | null>
```

Весь стан базується на сигналах. Компоненти використовують `computed()` для похідних значень.

---

## Потік WebSocket-повідомлень

```
+----------+                                          +-----------+
|  Angular |                                          | Go Backend|
|  Client  |                                          |           |
+----+-----+                                          +-----+-----+
     |                                                      |
     |  1. HTTP GET /ws (Upgrade: websocket)                |
     |----------------------------------------------------->|
     |  2. 101 Switching Protocols                          |
     |<-----------------------------------------------------|
     |                                                      |
     |  3. device.query.list  {requestId: "abc"}            |
     |----------------------------------------------------->|
     |  4. device.response.list {requestId: "abc"}          |
     |      (cached data, dataLoaded flags)                 |
     |<-----------------------------------------------------|
     |                                                      |
     |  5. device.data.updated (broadcast, progressive)     |
     |<-----------------------------------------------------|
     |  6. device.license.updated (broadcast)               |
     |<-----------------------------------------------------|
     |                                                      |
     |  7. ping                                             |
     |----------------------------------------------------->|
     |  8. pong                                             |
     |<-----------------------------------------------------|
     |                                                      |
     |  9. reload.trigger                                   |
     |----------------------------------------------------->|
     | 10. reload.response (ack to sender)                  |
     |<-----------------------------------------------------|
     | 11. reload.started (broadcast to all)                |
     |<=====================================================|
     | 12. reload.completed (broadcast to all)              |
     |<=====================================================|
     |                                                      |
```

---

## Конвеєр збірки

```
+-------------------+       +-------------------+       +------------------+
| Angular Build     |       | Go Build          |       | Output           |
|                   |       |                   |       |                  |
| frontend/src/ --->| npx   | cmd/wui/main.go   | go    |                  |
|   ng build        |------>| internal/app/     | build | bin/wui          |
|                   |       |   static/browser/ |------>| (single binary   |
| Output:           |       |   //go:embed      |       |  with embedded   |
| frontend/dist/ ---|copy|->|                   |       |  Angular SPA)    |
+-------------------+       +-------------------+       +------------------+
```

### Команди збірки

```bash
# Лише frontend
cd frontend && npx ng build

# Лише backend (потребує попередньо зібраного frontend)
go build ./cmd/wui

# Повна збірка
make build

# Розробка
make dev
```

---

## Зовнішні залежності

### Go

| Пакет | Призначення |
|---------|---------|
| `gorilla/websocket` | Протокол WebSocket (RFC 6455) |
| `go-chi/chi/v5` | HTTP-роутер |
| `cleanenv` | Конфігурація: YAML + env-змінні |
| `golang.org/x/text` | Кодування Windows-1251 |
| `gopkg.in/ini.v1` | Читання/запис INI-файлів |
| `google/uuid` | Генерація UUID |
| `log/slog` (stdlib) | Структуроване логування |

### Angular

| Пакет | Призначення |
|---------|---------|
| `@angular/core` | Фреймворк (v19+, standalone) |
| `@angular/router` | SPA-маршрутизація з lazy load |
| `rxjs` | Реактивні потоки (WebSocket) |
| Material Symbols | Шрифт іконок (outlined) |
| Montserrat / Commissioner | Типографіка |

---

## Підтримка платформ

```
+---------------------+----------------------------+
| Platform            | OLE/COM Support            |
+---------------------+----------------------------+
| Windows (production)| ole_windows.go (VBScript)  |
|                     | ole_native_windows.go (COM)|
+---------------------+----------------------------+
| macOS / Linux (dev) | ole_stub.go (no-op)        |
+---------------------+----------------------------+
```

Build-теги (`//go:build windows` / `//go:build !windows`) обирають правильну реалізацію під час компіляції.
