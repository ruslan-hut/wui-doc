# Специфікація протоколу WebSocket

**Версія**: 1.2
**Останнє оновлення**: 2026-03-27

## Огляд

WUI використовує **чисту архітектуру WebSocket** для всієї взаємодії клієнта із сервером. Усі запити даних, команди та оновлення в реальному часі проходять через єдине з'єднання WebSocket, що усуває потребу в HTTP REST endpoint'ах (окрім `/health` та обслуговування статичних файлів).

## Принципи архітектури

- **Єдине з'єднання**: одне постійне з'єднання WebSocket на клієнта
- **Двонаправленість**: повнодуплексна взаємодія (клієнт <-> сервер)
- **Кореляція запит-відповідь**: шаблон запиту зі співставленням за `requestId`
- **Прогресивне завантаження**: миттєва відповідь з кешованими даними + прогресивні оновлення
- **Пакетування повідомлень**: кілька JSON-повідомлень на один кадр WebSocket (розділені символами нового рядка)
- **Автоматичне перепідключення**: експоненційна затримка з 10 спробами повтору

## Формат конверта повідомлення

Усі повідомлення WebSocket використовують стандартний JSON-конверт:

```json
{
  "type": "device.query.list",
  "payload": { ... },
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "timestamp": "2026-01-31T12:00:00Z",
  "requestId": "abc-123",
  "error": {
    "code": "TIMEOUT",
    "message": "Query timeout after 10s",
    "details": {}
  }
}
```

### Поля

| Поле | Тип | Обов'язкове | Опис |
|-------|------|----------|-------------|
| `type` | `MessageType` | Так | Ідентифікатор типу повідомлення (див. типи нижче) |
| `payload` | `object` | Так | Дані, специфічні для повідомлення |
| `id` | `string` | Так | Унікальний UUID повідомлення (генерується автоматично) |
| `timestamp` | `string` | Так | Мітка часу ISO 8601 (генерується автоматично) |
| `requestId` | `string` | Ні | Кореляційний ідентифікатор для співставлення запит-відповідь |
| `error` | `ErrorInfo` | Ні | Деталі помилки (лише для відповідей з помилкою) |

### Структура ErrorInfo

```json
{
  "code": "TIMEOUT",
  "message": "Human-readable error message",
  "details": {}
}
```

### Коди помилок

| Код | Опис |
|------|-------------|
| `TIMEOUT` | Час очікування запиту або операції вичерпано |
| `NOT_FOUND` | Пристрій або ресурс не знайдено |
| `NO_DEVICES` | Не знайдено увімкнених пристроїв (пошук чека) |
| `QUERY_ERROR` | Збій запиту до бази даних |
| `DB_ERROR` | Помилка відкриття/читання/запису бази даних |
| `OLE_ERROR` | Збій операції OLE/COM |
| `RELOAD_IN_PROGRESS` | Перезавантаження вже виконується |
| `RELOAD_FAILED` | Збій перезавантаження |
| `REPORT_FAILED` | Збій генерації звіту |
| `ORDER_FAILED` | Збій замовлення ліцензії |
| `VALIDATION_ERROR` | Помилка валідації вхідних даних |
| `DUPLICATE_ERROR` | Дублікат запису (наприклад, назва форми оплати) |
| `LIMIT_ERROR` | Перевищено ліміт (наприклад, максимум 99 форм оплати) |
| `LOG_ERROR` | Помилка читання файлу журналу |
| `LAST_CHECK_ERROR` | Помилка розбору XML останнього чека |

Відповіді з помилкою використовують тип повідомлення `device.response.error` з `requestId` вихідного запиту.

---

## Типи повідомлень

### Запити (Клієнт -> Сервер)

#### `device.query.list`
Запит списку всіх пристроїв із кешованими даними.

**Payload запиту:**
```json
{
  "requestId": "req-uuid"
}
```

**Тип відповіді:** `device.response.list`
**Payload відповіді:**
```json
{
  "devices": [
    {
      "fiscalNumber": "4000167785",
      "index": 1,
      "enabled": true,
      "tin": "12345678",
      "dbPath": "C:\\path\\to\\db",
      "company": "Acme Corp",
      "state": "online",
      "offlineCount": 0,
      "offlineSince": null,
      "reservedNumbers": 100,
      "lastOfflineErr": "",
      "shiftStatus": "open",
      "shiftStart": "2026-01-31 08:00:00",
      "shiftId": 42,
      "error": null,
      "license": {
        "status": "active",
        "expiryDate": "2027-01-31",
        "licenseType": "full",
        "daysLeft": 365
      },
      "dataLoaded": true,
      "licenseLoaded": true,
      "orderDate": "2026-03-15",
      "keyDataAvailable": true
    }
  ]
}
```

**Прогресивні оновлення:**
Після початкової відповіді сервер транслює (broadcast):
- `device.data.updated` — коли завантажуються дані бази даних
- `device.license.updated` — коли завантажуються дані ліцензії

---

#### `device.query.payment-methods`
Отримати форми оплати для конкретного пристрою.

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.payment-methods`
```json
{
  "fiscalNumber": "4000167785",
  "paymentMethods": [
    { "id": 1, "formName": "Cash", "isCash": true },
    { "id": 2, "formName": "Card", "isCash": false }
  ]
}
```

---

#### `device.query.operators`
Отримати операторів для конкретного пристрою.

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.operators`
```json
{
  "fiscalNumber": "4000167785",
  "operators": [
    {
      "id": 1,
      "operatorName": "John Doe",
      "keyPath": "/path/to/key",
      "inn": "1234567890",
      "keyInfo": {
        "startKey": "1.1.2026 0:0:0",
        "endKey": "31.12.2026 23:59:59",
        "serial": "ABC123",
        "issuer": "CA Name",
        "daysLeft": 280,
        "status": "active"
      }
    }
  ]
}
```

`keyInfo` дорівнює `null`, якщо для оператора немає запису в `dat.ini`.

---

#### `device.query.shifts`
Отримати зміни для пристрою в межах діапазону дат.

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "from": "2026-01-01 00:00:00",
  "to": "2026-01-31 23:59:59",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.shifts`
```json
{
  "fiscalNumber": "4000167785",
  "shifts": [
    {
      "id": 123,
      "startDate": "2026-01-31 08:00:00",
      "endDate": "2026-01-31 20:00:00",
      "operatorName": "John Doe",
      "operatorId": "1",
      "lastLocalCheckNumber": 150
    }
  ]
}
```

---

#### `device.query.checks`
Отримати чеки для конкретної зміни.

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "shiftId": 123,
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.checks`
```json
{
  "fiscalNumber": "4000167785",
  "shiftId": 123,
  "checks": [
    {
      "id": 456,
      "documentType": 0,
      "checkNumber": 1,
      "fiscalNumber": "abc123xyz",
      "dateTime": "2026-01-31 10:30:00",
      "totalSum": 249.50,
      "operatorName": "John Doe",
      "isCanceled": false,
      "offlineStatus": 0,
      "mac": "ABCDEF1234567890"
    }
  ]
}
```

---

#### `device.query.check.search`
Пошук чека за його фіскальним номером серед усіх увімкнених пристроїв (або конкретного пристрою).

**Запит:**
```json
{
  "fiscalNumber": "abc123xyz",
  "deviceFiscalNumber": "4000167785",
  "requestId": "req-uuid"
}
```

`deviceFiscalNumber` є необов'язковим. Якщо його не вказано, пошук виконується серед усіх увімкнених пристроїв.

**Відповідь:** `device.response.check.search`
```json
{
  "check": {
    "id": 456,
    "documentType": 0,
    "checkNumber": 1,
    "fiscalNumber": "abc123xyz",
    "dateTime": "2026-01-31 10:30:00",
    "totalSum": 249.50,
    "operatorName": "John Doe",
    "isCanceled": false,
    "offlineStatus": 0,
    "mac": "ABCDEF1234567890",
    "shiftId": 123
  },
  "shift": {
    "id": 123,
    "startDate": "2026-01-31 08:00:00",
    "endDate": "2026-01-31 20:00:00",
    "operatorName": "John Doe",
    "operatorId": "1",
    "lastLocalCheckNumber": 150
  },
  "deviceFiscalNumber": "4000167785",
  "deviceCompany": "Acme Corp"
}
```

---

#### `device.query.check-preview`
Отримати рядки попереднього перегляду чека для конкретного чека через OLE/COM.

**Запит:**
```json
{
  "deviceFiscalNumber": "4000167785",
  "checkFiscalNumber": "abc123xyz",
  "documentType": 0,
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.check-preview`
```json
{
  "deviceFiscalNumber": "4000167785",
  "checkFiscalNumber": "abc123xyz",
  "documentType": 0,
  "lines": ["Line 1", "Line 2", "..."],
  "found": true,
  "error": ""
}
```

---

#### `device.query.settings`
Отримати налаштування пристрою (розібрані з settings.ini).

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.settings`
```json
{
  "fiscalNumber": "4000167785",
  "enabled": true,
  "offlineMin": 1,
  "offlineMax": 100,
  "acsksettings": 0,
  "useACSKTSPserver": false,
  "limitCertificate": 14,
  "fiscalMode": true,
  "ecoPrt": false,
  "automatPrintCheck": true,
  "allowableCash": 0,
  "printerWidth": 42,
  "printerName": "POS-58",
  "logOn": true,
  "shiftCloseTime": 0,
  "shiftCloseTimeEnabled": false,
  "shiftCashInOut": false
}
```

---

#### `device.query.log`
Прочитати фрагмент файлу журналу пристрою (log.txt або log_old.txt). Кодування Windows-1251 обробляється автоматично.

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "fileName": "log.txt",
  "offset": 0,
  "limit": 65536,
  "requestId": "req-uuid"
}
```

`limit` за замовчуванням дорівнює 65536 (64 КБ), якщо дорівнює 0 або не вказано.

**Відповідь:** `device.response.log`
```json
{
  "fiscalNumber": "4000167785",
  "content": "log text content...",
  "totalSize": 524288,
  "offset": 0,
  "bytesRead": 65536,
  "hasMore": true
}
```

`bytesRead` відстежує сирі байти файлу (а не довжину декодованого рядка) для коректного обчислення зміщення в наступних запитах.

---

#### `device.query.last-check`
Отримати розібрані дані останнього чека (з 000.xml).

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.last-check`
```json
{
  "found": true,
  "fiscalNumber": "abc123xyz",
  "taxNumber": "12345678",
  "docIndex": 1,
  "checkType": 0,
  "checkNumber": 42,
  "timestamp": "2026-01-31 10:30:00",
  "totalSum": 24950,
  "items": [
    {
      "lineNumber": 1,
      "code": "001",
      "name": "Product A",
      "sum": 12500,
      "quantity": 1000,
      "price": 12500,
      "taxGroup": 1
    }
  ],
  "discounts": [],
  "payments": [
    {
      "lineNumber": 1,
      "type": 0,
      "name": "Cash",
      "sum": 24950
    }
  ],
  "mac": "ABCDEF1234567890",
  "rawXml": ""
}
```

`rawXml` заповнюється лише тоді, коли документ не є розпізнаним чеком продажу/повернення. `items`, `discounts` та `payments` завжди є масивами (ніколи не `null`).

---

### Команди (Клієнт -> Сервер)

#### `reload.trigger`
Запустити перезавантаження backend'у (settings.ini, база даних, ліцензії).

**Запит:**
```json
{
  "requestId": "req-uuid"
}
```

**Відповідь:** `reload.response`
```json
{
  "reloadId": "reload-uuid",
  "message": "Reload completed successfully"
}
```

**Події broadcast:**
Сервер транслює всім клієнтам під час перезавантаження:
1. `reload.started` — перезавантаження починається
2. `device.data.updated` — оновлення бази даних для кожного пристрою
3. `device.license.updated` — оновлення ліцензії для кожного пристрою
4. `reload.completed` — перезавантаження завершено

---

#### `device.command.report-x`
Згенерувати X-звіт (денний звіт без обнулення).

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.report-x`
```json
{
  "fiscalNumber": "4000167785",
  "success": true,
  "lines": ["Report line 1", "Report line 2", "..."],
  "error": ""
}
```

---

#### `device.command.report-z`
Згенерувати Z-звіт (денний звіт з обнуленням, закриває зміну).

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.report-z`
```json
{
  "fiscalNumber": "4000167785",
  "success": true,
  "lines": ["Report line 1", "Report line 2", "..."],
  "error": ""
}
```

---

#### `device.command.report-period`
Згенерувати періодичний звіт за діапазон дат.

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "startDate": "2026-01-01",
  "endDate": "2026-01-31",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.report-period`
```json
{
  "fiscalNumber": "4000167785",
  "success": true,
  "lines": ["Report line 1", "Report line 2", "..."],
  "error": ""
}
```

---

#### `device.command.update-setting`
Оновити одне налаштування пристрою в settings.ini.

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "key": "EcoPrt",
  "value": "1",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.update-setting`
```json
{
  "fiscalNumber": "4000167785",
  "key": "EcoPrt",
  "value": "1",
  "success": true
}
```

---

#### `device.command.create-payment-method`
Створити нову форму оплати. Валідує: дублікат назви, ліміт максимум 99. `ISCASH` завжди дорівнює 0.

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "name": "New Payment Method",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.create-payment-method`
```json
{
  "fiscalNumber": "4000167785",
  "paymentMethods": [
    { "id": 1, "formName": "Cash", "isCash": true },
    { "id": 5, "formName": "New Payment Method", "isCash": false }
  ]
}
```

Повертає повний оновлений список форм оплати.

---

#### `device.command.update-payment-method`
Оновити назву наявної форми оплати. Редагувати можна лише методи з `id > 4` (перші 4 є системними за замовчуванням).

**Запит:**
```json
{
  "fiscalNumber": "4000167785",
  "id": 5,
  "name": "Updated Name",
  "requestId": "req-uuid"
}
```

**Відповідь:** `device.response.update-payment-method`
```json
{
  "fiscalNumber": "4000167785",
  "paymentMethods": [
    { "id": 1, "formName": "Cash", "isCash": true },
    { "id": 5, "formName": "Updated Name", "isCash": false }
  ]
}
```

Повертає повний оновлений список форм оплати.

---

#### `license.command.order`
Розмістити замовлення ліцензії для одного або кількох пристроїв.

**Запит:**
```json
{
  "fiscalNumbers": ["4000167785", "4001226137"],
  "email": "user@example.com",
  "requestId": "req-uuid"
}
```

**Відповідь:** `license.response.order`
```json
{
  "results": [
    {
      "tin": "12345678",
      "company": "Acme Corp",
      "fiscalNumbers": ["4000167785", "4001226137"],
      "success": true,
      "orderDate": "2026-03-27",
      "error": ""
    }
  ]
}
```

Результати згруповані за TIN (компанією). Кожна група показує, чи було замовлення успішним.

---

### Broadcast-повідомлення (Сервер -> Усі клієнти)

#### `device.data.updated`
Надсилається, коли дані бази даних пристрою завантажено/оновлено.

**Payload:**
```json
{
  "fiscalNumber": "4000167785",
  "company": "Acme Corp",
  "state": "online",
  "offlineCount": 0,
  "offlineSince": null,
  "reservedNumbers": 100,
  "lastOfflineErr": "",
  "shiftStatus": "open",
  "shiftStart": "2026-01-31 08:00:00",
  "shiftId": 42,
  "error": null,
  "loadedAt": "2026-01-31T12:00:00Z",
  "keyDataAvailable": true,
  "expiringKeyOperators": [
    {
      "operatorName": "John Doe",
      "inn": "1234567890",
      "daysLeft": 7,
      "endKey": "7.2.2026 23:59:59"
    }
  ]
}
```

`expiringKeyOperators` містить операторів, чиї ключі спливають протягом 14 днів (використовується для сповіщень).

---

#### `device.license.updated`
Надсилається, коли дані ліцензії пристрою завантажено/оновлено.

**Payload:**
```json
{
  "fiscalNumber": "4000167785",
  "license": {
    "status": "active",
    "expiryDate": "2027-01-31",
    "licenseType": "full",
    "daysLeft": 365,
    "error": null
  },
  "loadedAt": "2026-01-31T12:00:00Z"
}
```

---

#### `reload.started`
Транслюється, коли перезавантаження починається.

**Payload:**
```json
{
  "reloadId": "reload-uuid",
  "startedAt": "2026-01-31T12:00:00Z"
}
```

---

#### `reload.completed`
Транслюється, коли перезавантаження успішно завершується.

**Payload:**
```json
{
  "reloadId": "reload-uuid",
  "completedAt": "2026-01-31T12:00:05Z",
  "duration": 5000000000,
  "errors": [],
  "fiscalNumbers": ["4000167785", "4001226137"]
}
```

---

#### `reload.error`
Транслюється, коли перезавантаження завершується невдало.

**Payload:**
```json
{
  "reloadId": "reload-uuid",
  "error": "Reload timeout: context deadline exceeded",
  "message": "Reload failed"
}
```

---

### Повідомлення heartbeat

#### `ping` (Клієнт -> Сервер)
Клієнт надсилає ping кожні 30 секунд.

**Payload:** `{}`

#### `pong` (Сервер -> Клієнт)
Сервер відповідає на ping.

**Payload:** `{}`

---

## Шаблон запит-відповідь

### Потік запиту

```
1. Client generates requestId (UUID)
2. Client sends query message with requestId in payload
3. Client waits for response with matching requestId (Promise)
4. Server processes query with 10s timeout
5. Server sends response with same requestId
6. Client resolves Promise with response payload
```

`requestId` може бути або в payload, або в полі `requestId` конверта повідомлення. Сервер перевіряє обидва розташування (спочатку payload, потім конверт).

### Реалізація на frontend

```typescript
async query<T>(type: string, payload: any, timeoutMs: number = 15000): Promise<T> {
  const requestId = crypto.randomUUID();

  const responsePromise = firstValueFrom(
    this.messages$.pipe(
      filter(msg => msg.requestId === requestId),
      timeout(timeoutMs),
      take(1)
    )
  );

  this.send({ type, payload: { ...payload, requestId }, ... });

  const response = await responsePromise;

  if (response.error) {
    throw new Error(`${response.error.code}: ${response.error.message}`);
  }

  return response.payload as T;
}
```

---

## Пакетування повідомлень

### Оптимізація backend'у

Щоб зменшити накладні витрати на кадри WebSocket, backend пакетує кілька повідомлень з черги в один кадр, розділяючи їх символами нового рядка:

```
{"type":"device.data.updated","payload":{...},"id":"...","timestamp":"..."}
{"type":"device.license.updated","payload":{...},"id":"...","timestamp":"..."}
```

### Обробка на frontend

Frontend розділяє повідомлення за символами нового рядка та розбирає кожен JSON-об'єкт окремо:

```typescript
private handleMessage(event: MessageEvent): void {
  const data = event.data.trim();
  if (!data) return;

  const messages = data.split('\n').filter(line => line.trim());

  for (const msgStr of messages) {
    const message: WebSocketMessage = JSON.parse(msgStr);
    this.messageSubject.next(message);
  }
}
```

---

## Тайм-аути

| Операція | Тайм-аут | Обробник |
|-----------|---------|---------|
| Запит (frontend) | 15с | `wsService.query()` кидає помилку |
| Запит до бази даних (backend) | 10с | Повертає відповідь з помилкою `TIMEOUT` |
| OLE ліцензія/попередній перегляд (backend) | 10с | Повертає `TIMEOUT` або `OLE_ERROR` |
| Перезавантаження (backend) | 30с | Транслює `reload.error` |
| Запобіжник перезавантаження (frontend) | 45с | Показує помилку тайм-ауту, скидає стан завантаження |

---

## Прогресивне завантаження

### Потік початкового завантаження

```
1. Frontend: Send device.query.list
2. Backend: Return immediate response with:
   - Fiscal numbers (always present)
   - Cached database data (if available)
   - Cached license data (if available)
   - dataLoaded: true/false flags
   - licenseLoaded: true/false flags
3. Frontend: Display devices immediately, show skeletons for loading data
4. Backend: Broadcast device.data.updated as each device loads
5. Frontend: Update devices reactively, remove skeletons
6. Backend: Broadcast device.license.updated as each license loads
7. Frontend: Update license info reactively
```

### Опитування на предмет завершення

Frontend опитує кожні 2 секунди (максимум 30с), щоб перевірити стани завантаження:

```typescript
this.loadingPollInterval = setInterval(async () => {
  const elapsed = Date.now() - this.pollStartTime;

  if (elapsed >= 30000) {
    this.stopLoadingStatePoll();
    return;
  }

  if (!this.hasLoadingDevices()) {
    this.stopLoadingStatePoll();
    return;
  }

  await this.pollDeviceStates();
}, 2000);
```

---

## Керування з'єднанням

### Автоматичне перепідключення

Frontend реалізує експоненційну затримку:

**Послідовність затримок:** 3с -> 6с -> 12с -> 24с -> 30с (максимум)
**Максимум спроб:** 10

### Стани з'єднання

| Стан | Опис |
|-------|-------------|
| `connecting` | Виконується початкове з'єднання |
| `connected` | WebSocket відкрито, heartbeat активний |
| `disconnected` | З'єднання закрито (штатно або через мережеву помилку) |
| `error` | Досягнуто максимальної кількості спроб перепідключення |

---

## Безпека

### Поточний стан (локальний десктоп)

- Дозволено всі джерела (origins) (лише localhost)
- Автентифікація не потрібна
- Без обмеження частоти запитів

### Майбутнє (віддалений доступ)

- Валідація заголовка `Origin` за білим списком
- Використання `wss://` (WebSocket Secure поверх TLS)
- Реалізація автентифікації на основі токенів під час рукостискання
- Обмеження частоти запитів на клієнта (повідомлень/секунду)
- Обмеження розміру повідомлення (максимум 512 КБ)
- Валідація вхідних даних для всіх payload'ів

---

## Швидкий довідник

### Усі типи повідомлень

**Запити (Клієнт -> Сервер):**

| Тип | Тип відповіді |
|------|---------------|
| `device.query.list` | `device.response.list` |
| `device.query.payment-methods` | `device.response.payment-methods` |
| `device.query.operators` | `device.response.operators` |
| `device.query.shifts` | `device.response.shifts` |
| `device.query.checks` | `device.response.checks` |
| `device.query.check.search` | `device.response.check.search` |
| `device.query.check-preview` | `device.response.check-preview` |
| `device.query.settings` | `device.response.settings` |
| `device.query.log` | `device.response.log` |
| `device.query.last-check` | `device.response.last-check` |

**Команди (Клієнт -> Сервер):**

| Тип | Тип відповіді |
|------|---------------|
| `reload.trigger` | `reload.response` |
| `device.command.report-x` | `device.response.report-x` |
| `device.command.report-z` | `device.response.report-z` |
| `device.command.report-period` | `device.response.report-period` |
| `device.command.update-setting` | `device.response.update-setting` |
| `device.command.create-payment-method` | `device.response.create-payment-method` |
| `device.command.update-payment-method` | `device.response.update-payment-method` |
| `license.command.order` | `license.response.order` |

**Broadcast'и (Сервер -> Усі клієнти):**

| Тип | Коли |
|------|------|
| `device.data.updated` | Дані бази даних пристрою завантажено |
| `device.license.updated` | Ліцензію пристрою завантажено |
| `reload.started` | Перезавантаження почалося |
| `reload.completed` | Перезавантаження завершено |
| `reload.error` | Перезавантаження завершилося невдало |

**Heartbeat:**

| Тип | Напрямок |
|------|-----------|
| `ping` | Клієнт -> Сервер |
| `pong` | Сервер -> Клієнт |

**Помилка:**

| Тип | Опис |
|------|-------------|
| `device.response.error` | Загальна відповідь з помилкою з кореляцією за `requestId` |

---

## Тестування

### Ручне тестування за допомогою `wscat`

```bash
# Connect
wscat -c ws://localhost:8080/ws

# List devices
> {"type":"device.query.list","payload":{"requestId":"test-1"},"id":"msg-1","timestamp":"2026-01-31T12:00:00Z"}

# Get shifts
> {"type":"device.query.shifts","payload":{"fiscalNumber":"4000167785","from":"2026-01-01 00:00:00","to":"2026-01-31 23:59:59","requestId":"test-2"},"id":"msg-2","timestamp":"2026-01-31T12:00:00Z"}

# Trigger reload
> {"type":"reload.trigger","payload":{"requestId":"test-3"},"id":"msg-3","timestamp":"2026-01-31T12:00:00Z"}

# Send ping
> {"type":"ping","payload":{},"id":"msg-4","timestamp":"2026-01-31T12:00:00Z"}
```

---

## Усунення несправностей

### Клієнт застряг у стані завантаження

**Симптом:** UI безкінечно показує індикатор завантаження
**Причини:**
1. Перезавантаження backend'у зависло (перевірте журнали)
2. Відсутній broadcast `reload.completed`
3. Тайм-аут на frontend не спрацьовує

**Виправлення:** Frontend має 45-секундний запобіжний тайм-аут, який скидає стан

### Помилки розбору JSON

**Симптом:** `Unexpected non-whitespace character after JSON`
**Причина:** Frontend не обробляє пакетовані повідомлення
**Виправлення:** Розділяйте повідомлення за символами нового рядка перед розбором

### Дубльовані запуски перезавантаження

**Симптом:** Дві події `reload.started` для одного перезавантаження
**Причина:** Відповідь використовує неправильний тип повідомлення
**Виправлення:** Використовуйте `reload.response` для відповіді на запит, а не `reload.started`

---

**Історія версій протоколу:**
- **v1.2** (2026-03-27): Налаштування, звіти, замовлення ліцензій, пошук/попередній перегляд чеків, CRUD форм оплати
- **v1.1** (2026-01-31): Чиста архітектура WebSocket, пакетування повідомлень
- **v1.0** (2026-01-23): Початковий гібрид WebSocket + HTTP
