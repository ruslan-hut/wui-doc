# База даних користувачів WUI — посібник з інтеграції для Windows-застосунку

Цей документ описує **базу даних користувачів**, яку WUI (вебінтерфейс на Go) читає для
входу та авторизації. Очікується, що Windows-застосунок (WebCheck):

1. **Попередньо створює** користувача `admin` під час початкового налаштування (якщо такого ще немає).
2. **Встановлює / скидає** пароль будь-якого користувача безпосередньо в цій базі даних.

WUI і Windows-застосунок використовують **той самий файл SQLite** і **ті самі параметри
bcrypt**, тому хеші паролів, записані будь-якою зі сторін, є взаємозамінними.

---

## 1. Файл бази даних

| | |
|---|---|
| Рушій | SQLite 3 |
| Шлях за замовчуванням | `C:\ProgramData\WebCheck\users.db` |
| Джерело шляху | `<webcheck.path>\users.db` — `webcheck.path` з конфігурації WUI (`WC_PATH` env / `webcheck.path` YAML), за замовчуванням `C:\ProgramData\WebCheck` |

WUI створює файл і застосовує схему під час запуску, якщо його не існує
(`internal/service/auth/store.go:OpenStore`). Windows-застосунок також може створити його
за наведеною нижче схемою — `CREATE TABLE IF NOT EXISTS` робить цю операцію ідемпотентною з
обох сторін.

> Це **окрема** база даних від БД даних пристроїв і від `data/wui.db`.
> Вона містить лише користувачів, дозволи та сесії.

### Паралельний доступ

Два процеси пишуть в один файл, тому завжди відкривайте його з **busy timeout** і
використовуйте WAL, щоб зменшити кількість помилок `database is locked`:

```sql
PRAGMA busy_timeout = 5000;
PRAGMA journal_mode = WAL;
```

Тримайте транзакції запису короткими. WUI відкриває БД з одним з'єднанням і
часом життя з'єднання 5 с.

---

## 2. Схема

Авторитетне джерело: `internal/service/auth/store.go` (константа `schemaSQL`).

```sql
CREATE TABLE IF NOT EXISTS users (
    id            INTEGER PRIMARY KEY AUTOINCREMENT,
    login         TEXT NOT NULL UNIQUE COLLATE NOCASE,
    name          TEXT NOT NULL,
    password_hash TEXT NOT NULL,
    role          TEXT NOT NULL CHECK(role IN ('admin','user')),
    email         TEXT,
    phone         TEXT,
    notes         TEXT,
    disabled      INTEGER NOT NULL DEFAULT 0,
    created_at    TEXT NOT NULL,
    updated_at    TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS user_device_permissions (
    user_id       INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    fiscal_number TEXT NOT NULL,
    section       INTEGER NOT NULL CHECK(section IN (1,2,3,4)),
    PRIMARY KEY (user_id, fiscal_number, section)
);
CREATE INDEX IF NOT EXISTS idx_perm_user ON user_device_permissions(user_id);

CREATE TABLE IF NOT EXISTS sessions (
    token        TEXT PRIMARY KEY,
    user_id      INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    created_at   TEXT NOT NULL,
    expires_at   TEXT NOT NULL,
    last_seen_at TEXT NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_sessions_user ON sessions(user_id);
```

### Стовпці таблиці `users`

| Стовпець | Тип | Правила |
|---|---|---|
| `id` | INTEGER | Автоінкрементний PK. Не встановлюйте його вручну. |
| `login` | TEXT | **Обов'язковий, унікальний, нечутливий до регістру** (`COLLATE NOCASE`). `Admin` == `admin`. |
| `name` | TEXT | **Обов'язковий.** Відображуване ім'я. |
| `password_hash` | TEXT | **Обов'язковий.** Хеш bcrypt — див. §3. Ніколи не зберігайте відкритий текст. |
| `role` | TEXT | **Обов'язковий.** Рівно `'admin'` або `'user'` (контролюється CHECK). |
| `email` | TEXT | Необов'язковий (може бути NULL). |
| `phone` | TEXT | Необов'язковий (може бути NULL). |
| `notes` | TEXT | Необов'язковий (може бути NULL). |
| `disabled` | INTEGER | `0` = активний, `1` = заблокований для входу. За замовчуванням `0`. |
| `created_at` | TEXT | **Обов'язковий.** Рядок локального часу — див. §5. |
| `updated_at` | TEXT | **Обов'язковий.** Рядок локального часу — див. §5. |

> **Зовнішні ключі:** щоб `ON DELETE CASCADE` для дозволів/сесій
> справді спрацьовував, виконуйте `PRAGMA foreign_keys = ON;` на кожному з'єднанні (за
> замовчуванням у SQLite це OFF). WUI покладається на це; Windows-застосунок також має це робити.

---

## 3. Хешування паролів (КРИТИЧНО)

Хеші створюються та перевіряються за допомогою **bcrypt** і мають використовувати **однаковий cost**
з обох сторін, щоб дайджести були взаємозамінними.

| | |
|---|---|
| Алгоритм | bcrypt |
| Cost | **12** (`auth.BcryptCost`, `internal/service/auth/hash.go`) |
| Бібліотека Go | `golang.org/x/crypto/bcrypt` |
| Формат зберігання | Повний рядок bcrypt, напр. `$2a$12$....` (60 символів) — зберігайте дослівно у `password_hash` |

Верифікатор Go приймає префікси `$2a$`, `$2b$` та `$2y$`, тому стандартна
бібліотека bcrypt для .NET / C++ є сумісною. **Використовуйте cost 12.**

**Приклад для .NET (BCrypt.Net-Next):**
```csharp
string hash = BCrypt.Net.BCrypt.HashPassword(plainPassword, workFactor: 12);
// store `hash` in password_hash
```

**Не** вигадуйте окремий стовпець для солі — bcrypt вбудовує сіль у сам рядок хешу.

### Політика паролів

WUI застосовує цю політику, коли *він сам* встановлює пароль
(`internal/service/auth/password.go`). Windows-застосунок має застосовувати ті самі
правила, щоб користувачі згодом могли змінити свій пароль через WUI:

- Рівно **10 символів**
- Щонайменше одна **мала** літера, одна **велика** літера, одна **цифра**
- **Лише літери та цифри** — без пробілів і спеціальних символів

(Сама БД не контролює довжину/складність — вона лише зберігає хеш —
але невідповідний пароль буде відхилено, якщо користувач спробує змінити його у WUI.)

---

## 4. Операції, які виконує Windows-застосунок

### 4a. Попереднє створення admin під час початкового налаштування

Створюйте admin **лише якщо таблиця порожня**. WUI показує екран початкового
налаштування (bootstrap), коли `SELECT COUNT(*) FROM users = 0`; якщо ви попередньо створите admin, цей екран
пропускається, і користувач входить у звичайний спосіб.

```sql
-- Run once, only when no users exist yet.
INSERT INTO users (login, name, password_hash, role, disabled, created_at, updated_at)
SELECT 'admin', 'Адміністратор', :bcrypt_hash, 'admin', 0,
       strftime('%Y-%m-%d %H:%M:%S','now','localtime'),
       strftime('%Y-%m-%d %H:%M:%S','now','localtime')
WHERE NOT EXISTS (SELECT 1 FROM users);
```

- `:bcrypt_hash` = bcrypt (cost 12) обраного пароля адміністратора.
- `role` **має** бути `'admin'`. Адміністратори обходять усі перевірки дозволів на пристрої.
- `email`, `phone`, `notes` можна не вказувати (NULL).

### 4b. Встановлення / скидання пароля користувача

```sql
UPDATE users
   SET password_hash = :bcrypt_hash,
       updated_at    = strftime('%Y-%m-%d %H:%M:%S','now','localtime')
 WHERE login = :login COLLATE NOCASE;   -- or WHERE id = :id

-- REQUIRED: invalidate the user's active sessions so the old password's
-- tokens can no longer be used.
DELETE FROM sessions
 WHERE user_id = (SELECT id FROM users WHERE login = :login COLLATE NOCASE);
```

> **Завжди видаляйте сесії користувача при зміні його пароля.** Інакше
> будь-яка сесія, створена до скидання, залишається дійсною до 24 год. Це повторює
> те, що WUI робить у `UpdatePasswordHash`.

### 4c. Вимкнення / увімкнення користувача (необов'язково)

```sql
UPDATE users SET disabled = 1, updated_at = strftime('%Y-%m-%d %H:%M:%S','now','localtime')
 WHERE login = :login COLLATE NOCASE;   -- 0 to re-enable
-- When disabling, also clear sessions (same DELETE as 4b).
```

---

## 5. Формат дати / часу (КРИТИЧНО)

Усі стовпці з мітками часу — це **рядки локального часу** у форматі `YYYY-MM-DD HH:MM:SS`,
**без часового поясу, не UTC, не Unix epoch**. Приклад: `2026-05-28 14:30:45`.

Це відповідає решті рівня даних WUI/WebCheck. Генеруйте їх за допомогою
`strftime('%Y-%m-%d %H:%M:%S','now','localtime')` (як вище) або вашого локального
`DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss")`. **Не** записуйте ISO-8601 з
`T`/`Z` і не зберігайте UTC — це призведе до відображення зі зсувом.

---

## 6. Таблиці, до яких Windows-застосунку НЕ слід писати

- **`sessions`** — повністю керується WUI (випадкові 64-символьні hex-токени, ковзний термін
  дії 24 год). Єдина дозволена операція — очищення `DELETE ... WHERE user_id = ?`,
  показане в §4b/§4c. Ніколи не вставляйте сесії.
- **`user_device_permissions`** — доступ до окремих пристроїв для користувачів без прав admin, керується
  адміністратором усередині WUI. Розділи: `1` продажі/зміни/чеки, `2` звіти,
  `3` налаштування/журнал/резервна копія/ліцензія, `4` оператори/способи оплати. Адміністратори ігнорують
  цю таблицю (повний доступ). Для випадків зі зміною пароля Windows-застосунку
  не потрібно її чіпати; залиште її WUI, якщо це явно не вимагається.

---

## 7. Швидкий довідник

| Елемент | Значення |
|---|---|
| Файл | `C:\ProgramData\WebCheck\users.db` |
| Хеш | bcrypt, **cost 12**, зберігайте повний рядок `$2*$12$...` |
| Ролі | `'admin'`, `'user'` (обмеження CHECK) |
| Логін | унікальний, нечутливий до регістру |
| `disabled` | `0` активний / `1` заблокований |
| Мітки часу | локальний `YYYY-MM-DD HH:MM:SS` |
| Політика паролів | 10 символів, велика+мала+цифра, лише літери та цифри |
| При зміні пароля | `UPDATE password_hash` **+** `DELETE FROM sessions` для цього користувача |
| PRAGMA з'єднання | `foreign_keys=ON`, `busy_timeout=5000`, `journal_mode=WAL` |
