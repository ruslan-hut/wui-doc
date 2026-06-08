# Карта коду

Де розташована кожна частина інтеграції WebCheck.

## Backend

| Файл | Відповідальність |
|------|----------------|
| `pkg/olexml/olexml.go` | Білдери вхідного XML (один на метод). |
| `internal/service/ole_interface.go` | Контракт інтерфейсу `OLEClient`. |
| `internal/service/ole_windows.go` | Windows-клієнт OLE — VBS-воркер, допоміжні функції диспетчеризації, парсери відповідей, вбудований `oleWorkerVBS`. |
| `internal/service/ole_native_windows.go` | Нативний шлях go-ole (лише `GetCheckPreview` його використовує; вимкнено за замовчуванням). |
| `internal/service/ole_stub.go` | Заглушка `WindowsOLEClient` для збірок не під Windows. |
| `internal/service/ole_mock.go` | Mock-клієнт для розробки не під Windows (повертає правдоподібні фейкові дані). |
| `internal/service/ole_service.go` | Обгортка `OLEService` — таймаути, кешування даних ліцензії, кеш попереднього перегляду. |
| `internal/service/ole_errors.go` | Sentinel-типи помилок (наприклад, `ErrOLENotReady`). |
| `internal/service/websocket_adapters.go` | Адаптери, що з'єднують `OLEService` з інтерфейсами пакета WebSocket (уникають циклу імпортів). |

## WebSocket

| Файл | Відповідальність |
|------|----------------|
| `internal/websocket/message.go` | Константи типів повідомлень і структури корисного навантаження запитів/відповідей. |
| `internal/websocket/query_handler.go` | Обробники запитів (cloud URL, баланс, попередній перегляд, список звітів, …). |
| `internal/websocket/command_handler.go` | Обробники команд (звіти, send-message, reload). |
| `internal/websocket/message_handler.go` | Switch, що маршрутизує типи повідомлень до обробників. |

## Frontend

| Файл | Відповідальність |
|------|----------------|
| `frontend/src/app/core/services/device.service.ts` | Методи WebSocket-клієнта (один на кожну backend-операцію). |
| `frontend/src/app/core/models/settings.model.ts` | TypeScript-інтерфейси, що відповідають backend DTO. |
| `frontend/src/app/features/checks/checks.component.{ts,html,scss}` | UI попереднього перегляду чека — кнопки Лінк ПРРО / Надіслати / PDF та вбудовані панелі. |
| `frontend/src/styles/_action-panel.scss` | Спільні стилі для вбудованих панелей дій (завантажуються глобально, щоб дотриматися бюджету CSS на компонент). |

## Приклад трасування: cloud URL

1. Користувач натискає "Лінк ПРРО" у `checks.component.html`
2. `requestCloudUrl()` → `DeviceService.getCheckCloudUrl(deviceFN, checkFN, docType)`
3. Надсилається WebSocket-повідомлення `device.query.check-cloud-url`
4. `MessageHandler` маршрутизує до `QueryHandler.HandleCheckCloudURLQuery`
5. → `OLEServiceAdapter.GetCheckCloudURL` → `OLEService.GetCheckCloudURL`
6. → `WindowsOLEClient.GetCheckCloudURL` будує XML через
   `olexml.GetCheckCloudurlXML`
7. Виклик VBS-воркера: `GETCLOUDURL <xml>` → `OK=True`
8. Виклик VBS-воркера: `STATUSBAR <xml>` → `HEX=...` (декодований XML)
9. `parseCheckCloudURLStatusXML` витягує `URL`
10. Надсилається відповідь `device.response.check-cloud-url` з `{url, found, error}`
11. Frontend відображає URL в полі вводу + кнопка копіювання
