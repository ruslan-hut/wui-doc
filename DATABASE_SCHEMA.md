# Документація схеми бази даних

> Згенеровано автоматично з прикладу бази даних: `.db/4000039540.db`
> Останнє оновлення: 2026-02-03

## Огляд

База даних POS використовує SQLite та зберігає інформацію про фіскальні чеки (checks), зміни, операторів, форми оплати та об'єкти оподаткування. База даних відповідає українським фіскальним вимогам.

---

## Основні таблиці

### 1. ksef (Чеки/Квитанції)

Головна таблиця, що зберігає всі фіскальні документи (чеки, операції зі змінами, звіти).

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Унікальний ідентифікатор документа |
| `checkid` | TEXT | Внутрішній ID чека |
| `checkxml` | TEXT | XML-подання чека |
| `checksigned` | TEXT | Цифрово підписані дані чека |
| `signedanswerfromficscal` | TEXT | Відповідь від фіскального сервера |
| `checkidficscal` | TEXT | **Фіскальний номер** (унікальний ідентифікатор від податкової) |
| `localchecknumber` | INTEGER | Локальний послідовний номер чека |
| `DocType` | INTEGER | Тип документа (8=відкриття зміни, 80=закриття зміни тощо) |
| `sum` | DECIMAL(17,2) | Загальна сума |
| `mac` | TEXT | Код автентифікації повідомлення |
| `shiftid` | INTEGER | Посилання на SHIFTS.ID |
| `dt` | DATETIME | Часова мітка документа |
| `offline` | INTEGER | Статус офлайн (0=онлайн, 1=в очікуванні, 2=надіслано офлайн, 3=помилка) |

**Індекси:**
- `checkidficscalind` на `checkidficscal` ⚡ (використовується для пошуку)
- `offlineind` на `offline`
- `shiftidind` на `shiftid`
- `DocTypeind` на `DocType`
- `checkidind` на `checkid`

**Ключові зв'язки:**
- `shiftid` → `SHIFTS.ID` (до якої зміни належить цей чек)

**Важливі примітки:**
- ⚠️ Назва стовпця — `checkidficscal` (не `checkidfiscal` — має незвичне написання)
- Використовуйте цю таблицю для пошуку чека за фіскальним номером
- `DocType = 8` = відкриття зміни, `DocType = 80` = закриття зміни

---

### 2. SHIFTS

Зберігає інформацію про зміни (робочі періоди касирів).

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Унікальний ідентифікатор зміни |
| `SHIFTID` | INTEGER | Послідовний номер зміни |
| `DATEBEG` | DATETIME | Дата/час початку зміни |
| `DATEEND` | DATETIME | Дата/час завершення зміни (NULL, якщо відкрита) |
| `ODRFO` | VARCHAR(12) | Податковий номер відповідальної особи |
| `ONAME` | VARCHAR(200) | Ім'я оператора |
| `TAXTIN` | VARCHAR(12) | Податковий ідентифікаційний номер |
| `TAXNAME` | VARCHAR(300) | Назва платника податків |
| `RROFISCAL` | BIGINT | Фіскальний номер РРО |
| `RROLOCAL` | BIGINT | Локальний номер РРО |
| `OPERATORID` | INTEGER | Посилання на OPERATORS.ID |
| `LASTFISCALCHECKNUMBER` | INTEGER | Останній фіскальний номер чека |
| `LastLocalCheckNumber` | INTEGER | Останній локальний номер чека |

**Індекси:**
- `shiftuniq` (UNIQUE) на `DATEEND`

**Ключові зв'язки:**
- `OPERATORID` → `OPERATORS.ID`
- На неї посилається `ksef.shiftid`

**Важливі примітки:**
- `DATEEND = "NULL"` (рядок) означає, що зміна наразі відкрита
- Назви стовпців у ВЕРХНЬОМУ РЕГІСТРІ (на відміну від таблиці ksef)

---

### 3. OPERATORS

Зберігає інформацію про касирів/операторів.

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Унікальний ідентифікатор оператора |
| `OPERATORNAME` | VARCHAR(256) | Повне ім'я оператора |
| `KEYPATH` | VARCHAR(256) | Шлях до ключа цифрового підпису |
| `KEYPASS` | VARCHAR(256) | Пароль ключа (зашифрований) |
| `INN` | VARCHAR(256) | Індивідуальний податковий номер |

**Ключові зв'язки:**
- На неї посилається `SHIFTS.OPERATORID`

---

### 4. PayForms (Форми оплати)

Типи форм оплати, доступні в системі.

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Ідентифікатор форми оплати |
| `NAME` | VARCHAR(256) | Назва форми оплати (наприклад, "Готівка", "Картка") |
| `ISCASH` | INTEGER | 1 = готівка, 0 = безготівка |

**Ключові зв'язки:**
- На неї посилається `CHECKPAY.PAYMENTFORM` (за назвою, не за ID)

---

### 5. TAXOBJECTS

Інформація про організацію та точку продажу.

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Ідентифікатор об'єкта оподаткування |
| `FN` | INTEGER | Фіскальний номер пристрою |
| `TIN` | VARCHAR(16) | Податковий ідентифікаційний номер |
| `INN` | INTEGER | Індивідуальний ідентифікаційний номер |
| `POINTNAME` | VARCHAR(256) | Назва точки продажу |
| `ORGNAME` | VARCHAR(256) | Назва організації |
| `POINTADDR` | VARCHAR(256) | Адреса точки продажу |

**Важливі примітки:**
- Використовуйте `ORGNAME` для відображення назви компанії/організації
- Один запис на один фіскальний пристрій

---

### 6. fns (Резерв фіскальних номерів)

Пул зарезервованих фіскальних номерів (виділених податковою).

| Column | Type | Опис |
|--------|------|------|
| `id` | INTEGER (PK, AUTO) | Ідентифікатор запису |
| `checkidfiscal` | TEXT | Фіскальний номер |
| `used` | DATETIME | Коли номер було використано (NULL, якщо не використано) |
| `added` | DATETIME | Коли додано до пулу |
| `sourceid` | TEXT | Ідентифікатор джерела |

**Важливі примітки:**
- Використовується для відстеження доступних фіскальних номерів
- `used IS NULL` = доступні номери
- Кількість значень NULL = кількість зарезервованих номерів, що залишилися

---

## Таблиці деталей чека

Ці таблиці зберігають детальну інформацію про вміст чеків.

### 7. CHECKHEAD (Заголовки чеків)

Основна інформація про чек та підсумки.

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Ідентифікатор заголовка чека |
| `SHIFTID` | INTEGER | Посилання на зміну |
| `UID` | VARCHAR(36) | Універсальний унікальний ідентифікатор |
| `DOCTYPE` | INT | Тип документа |
| `VER` | INT | Версія формату |
| `TIN` | VARCHAR(10) | Податковий номер |
| `INN` | VARCHAR(12) | Індивідуальний номер |
| `ORGNAME` | VARCHAR(256) | Назва організації |
| `POINTNAME` | VARCHAR(256) | Назва точки |
| `POINTADDR` | VARCHAR(256) | Адреса точки |
| `ORDERDATE` | DATETIME | Дата замовлення |
| `ORDERNUM` | TEXT | Номер замовлення |
| `ORDERTAXNUM` | TEXT | Податковий номер замовлення |
| `CASHDESKNUM` | BIGINT | Номер каси |
| `FN` | BIGINT | Фіскальний номер |
| `CASHIER` | VARCHAR(128) | Ім'я касира |
| `TOTALSUM` | DECIMAL(17,2) | Загальна сума чека |

**Ключові зв'язки:**
- `SHIFTID` → `SHIFTS.ID`
- `ID` → На нього посилаються CHECKBODY, CHECKPAY, CHECKTAX, CHECKEXCISE

---

### 8. CHECKBODY (Позиції чека)

Окремі товари/послуги в чеку.

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Ідентифікатор позиції |
| `CHECKID` | INTEGER | Посилання на CHECKHEAD.ID |
| `CODE` | VARCHAR(64) | Код товару |
| `UKTZED` | VARCHAR(15) | Код української класифікації (УКТЗЕД) |
| `GOODSNAME` | VARCHAR(128) | Назва товару/послуги |
| `UNITCODE` | VARCHAR(128) | Код одиниці виміру |
| `UNITNAME` | VARCHAR(64) | Назва одиниці виміру |
| `AMOUNT` | DECIMAL(17,3) | Кількість |
| `PRICE` | DECIMAL(17,2) | Ціна за одиницю |
| `LETTER` | VARCHAR(1) | Літерний код податку |
| `COST` | DECIMAL(17,2) | Загальна вартість (AMOUNT × PRICE) |
| `LETTEREXCISE` | VARCHAR(1) | Літерний код акцизу |

---

### 9. CHECKPAY (Оплати за чеком)

Форми оплати, використані в чеку (може бути декілька на один чек).

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Ідентифікатор запису оплати |
| `CHECKID` | INTEGER | Посилання на CHECKHEAD.ID |
| `PAYMENTFORM` | VARCHAR(64) | Назва форми оплати |
| `TOTALSUM` | DECIMAL(17,2) | Сума, сплачена цим способом |

---

### 10. CHECKTAX (Розбивка податків за чеком)

Розрахунки податків на чек.

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Ідентифікатор запису податку |
| `CHECKID` | INTEGER | Посилання на CHECKHEAD.ID |
| `TAXCODE` | VARCHAR(3) | Код податку (наприклад, "А", "Б", "В") |
| `TAXPRC` | DECIMAL(17,2) | Відсоток податку |
| `TAXSUM` | DECIMAL(17,2) | Сума податку |

---

### 11. CHECKEXCISE (Акцизний податок за чеком)

Інформація про акцизний податок (алкоголь, тютюн тощо).

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Ідентифікатор запису акцизу |
| `CHECKID` | INTEGER | Посилання на CHECKHEAD.ID |
| `EXCISECODE` | VARCHAR(64) | Код акцизу |
| `EXCISEPRC` | DECIMAL(17,2) | Відсоток акцизу |
| `EXCISESUM` | DECIMAL(17,2) | Сума акцизу |

---

## Таблиці конфігурації

### 12. TAXES

Визначення податкових ставок.

| Column | Type | Опис |
|--------|------|------|
| `ID` | INTEGER (PK, AUTO) | Ідентифікатор податку |
| `NAME` | TEXT | Назва податку |
| `EXCISE` | INTEGER | Чи є акцизним податком (1=так, 0=ні) |
| `TAXPRC` | DECIMAL(17,2) | Відсоток податку |

---

## Службові таблиці

### 13. Sessions

Управління сесіями (реалізація не показана у схемі).

### 14. backuplog

Журнал резервного копіювання/реплікації (автоматично заповнюється тригерами).

---

## Типові шаблони запитів

### Пошук чека за фіскальним номером

```sql
SELECT
    k.id, k.doctype, k.localchecknumber, k.checkidficscal,
    k.dt, k.sum, k.offline, k.shiftid,
    s.ID, s.DATEBEG, s.DATEEND, s.ONAME, s.OPERATORID, s.LASTLOCALCHECKNUMBER
FROM ksef k
INNER JOIN SHIFTS s ON k.shiftid = s.ID
WHERE k.checkidficscal = 'FISCAL_NUMBER_HERE'
LIMIT 1;
```

### Отримання чеків за зміну

```sql
SELECT id, doctype, localchecknumber, checkidficscal, dt, sum, offline
FROM ksef
WHERE shiftid = SHIFT_ID_HERE
ORDER BY dt ASC;
```

### Отримання відкритих змін

```sql
SELECT ID, DATEBEG, ONAME, OPERATORID
FROM SHIFTS
WHERE DATEEND = 'NULL'
ORDER BY DATEBEG DESC;
```

### Отримання змін за діапазон дат

```sql
SELECT ID, DATEBEG, DATEEND, ONAME, OPERATORID, LASTLOCALCHECKNUMBER
FROM SHIFTS
WHERE DATEBEG >= 'YYYY-MM-DD' AND DATEBEG <= 'YYYY-MM-DD'
ORDER BY DATEBEG DESC;
```

### Отримання назви організації

```sql
SELECT ORGNAME
FROM TAXOBJECTS
WHERE FN = FISCAL_NUMBER_HERE
LIMIT 1;
```

### Підрахунок офлайн-чеків

```sql
SELECT COUNT(id) FROM ksef WHERE offline = 2;
```

### Підрахунок зарезервованих фіскальних номерів

```sql
SELECT COUNT(id) FROM fns WHERE used IS NULL;
```

### Отримання повних деталей чека

```sql
-- Header
SELECT * FROM CHECKHEAD WHERE ID = CHECK_ID;

-- Line items
SELECT * FROM CHECKBODY WHERE CHECKID = CHECK_ID ORDER BY ID;

-- Payments
SELECT * FROM CHECKPAY WHERE CHECKID = CHECK_ID ORDER BY ID;

-- Taxes
SELECT * FROM CHECKTAX WHERE CHECKID = CHECK_ID ORDER BY ID;

-- Excise
SELECT * FROM CHECKEXCISE WHERE CHECKID = CHECK_ID ORDER BY ID;
```

---

## Важливі примітки

### Регістр назв стовпців

⚠️ **Зверніть увагу на непослідовність регістру:**
- Таблиця `ksef`: назви стовпців у **нижньому регістрі** (`checkidficscal`, `dt`, `offline` тощо)
- Таблиця `SHIFTS`: назви стовпців у **ВЕРХНЬОМУ РЕГІСТРІ** (`DATEBEG`, `DATEEND`, `ONAME` тощо)
- Інші таблиці: змішаний регістр

### Стовпець фіскального номера

⚠️ **Стовпець фіскального номера має незвичне написання:**
- Назва стовпця: `checkidficscal` (не `checkidfiscal`)
- Це НЕ помилка — це фактична назва стовпця в базі даних

### Статус зміни

- Відкрита зміна: `SHIFTS.DATEEND = "NULL"` (рядок "NULL", а не SQL NULL)
- Закрита зміна: `SHIFTS.DATEEND` містить фактичну дату/час

### Значення статусу офлайн

У `ksef.offline`:
- `0` = Онлайн (надіслано на фіскальний сервер)
- `1` = В очікуванні
- `2` = Офлайн (надіслано локально, ще не синхронізовано)
- `3` = Помилка

### Типи документів

У `ksef.DocType`:
- `8` = Відкриття зміни
- `80` = Закриття зміни
- Інші значення = звичайні чеки/документи

---

## Пов'язані файли

- Приклади баз даних: `/.db/*.db`
- Реалізація читача бази даних: `/internal/service/database_reader.go`
- Обробники WebSocket-запитів: `/internal/websocket/query_handler_database.go`
- Моделі фронтенду: `/frontend/src/app/core/models/settings.model.ts`

---

## Команда вилучення схеми

Щоб дослідити файл бази даних:

```bash
# List all tables
sqlite3 database.db ".tables"

# Get schema for a specific table
sqlite3 database.db ".schema TABLE_NAME"

# Get schema for all tables
sqlite3 database.db ".schema"

# Query data
sqlite3 database.db "SELECT * FROM ksef LIMIT 5;"
```
