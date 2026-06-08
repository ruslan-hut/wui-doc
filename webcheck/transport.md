# Transport

Усі COM-виклики проходять через стійкий VBScript-воркер (`cscript.exe`, що
виконує `oleWorkerVBS`, вбудований у `internal/service/ole_windows.go`).
Воркер підтримує COM-об'єкт активним між викликами.

Ми використовуємо міст VBS замість нативного go-ole, бо
`IDispatch.Invoke` go-ole у COM DLL WebCheckServer спричиняв
ACCESS_VIOLATION. Нативна COM-автоматизація VBScript обробляє ті самі
виклики без аварійного завершення. Нативний шлях зберігається лише для
`GetCheckPreview` (вимкнено за замовчуванням — див.
`ole_native_windows.go`).

## Двокрокова схема

Кожен задокументований метод (окрім самого `GetCheck`, який є однокроковим)
використовує:

1. **Надіслати вхідний XML методу** — драйвер повертає булеве значення
   (`OK=True` / `OK=False`).
2. **Надіслати `StatusBarXML(<deviceXML>)`** — драйвер повертає структурований
   результуючий XML, що несе фактичні дані.

`<deviceXML>` — це тривіальне корисне навантаження
`<InputParameters><Parameters FN="..."/>`, побудоване `olexml.DeviceXML`.

## Кодування

COM-драйвер повертає текст як UTF-8 або Windows-1251 залежно від виклику.
Бік Go автовизначає для кожного виклику:

```go
if !utf8.Valid(data) {
    data, _ = charmap.Windows1251.NewDecoder().Bytes(data)
}
```

Див. `parseStatusBarXML` у `ole_windows.go`.

## Hex-транспорт

VBS-воркер кодує вивід `StatusBarXML` у hex (4 hex-цифри на кожен
Unicode-руну) та додає префікс `HEX=`. Це необхідно, бо кодова сторінка
консолі Windows пошкоджує кирилицю, коли текст проходить через stdout. Бік
Go декодує за допомогою `decodeHexUTF16`.

Протокол виводу воркера:

```
READY                       <- on startup
OK=True | OK=False          <- after each Boolean-returning call
HEX=0420041F0420041E        <- after StatusBarXML (hex-encoded UTF-16 chars)
ERR=<message>               <- on COM error
<<<EOD>>>                   <- record separator
```

## Конкурентність

COM-драйвер не потоково-безпечний. `WindowsOLEClient.mu` (`sync.Mutex`)
серіалізує кожен виклик воркера. Якщо процес воркера аварійно завершується,
наступний виклик автоматично перезапускає його через `ensureWorker`.

## Таймаути

- Типовий таймаут на виклик: `s.timeout` (налаштовується в `OLEService`,
  зазвичай кілька секунд).
- X/Z/Period-звіти: `s.reportTimeout` (довший — фіскальне підписання та
  друк можуть займати 30–60 с).

Тривалі виклики використовують `sendCommandCtx`, який завершує воркер, якщо
контекст спливає (тож завислий COM-виклик не блокує назавжди).
