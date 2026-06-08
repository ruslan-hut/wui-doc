# Звіти — ReportX, ReportZ, GetPeriodReport

Стандартні фіскальні звіти. Кожен є двокроковим викликом (`<Method>(xml)`,
потім `StatusBarXML(xml)`); рядки чека повертаються в тій самій формі
атрибутів `L0..LN`, що й у [`GetCheck`](get-check.md).

Звіти займають помітно більше часу, ніж інші виклики (фіскальне підписання +
друк можуть тривати 30–60 с), тож сервіс використовує `s.reportTimeout`
замість типового `s.timeout`.

Джерело: `internal/service/ole_windows.go:executeReport`,
`doReportCall` (спільний двокроковий конвеєр + парсинг).

## ReportX — підсумок зміни лише для читання

Не закриває зміну. Білдер: `olexml.DeviceXML(fn)` (лише `FN`).

```xml
<InputParameters><Parameters FN="4000167785" /></InputParameters>
```

## ReportZ — закриття зміни

Створює фіскальний документ закриття зміни. Білдер: `olexml.ReportZXML`.

```xml
<InputParameters><Parameters FN="4000167785" Operatorid="" /></InputParameters>
```

`Operatorid` навмисно порожній — драйвер обирає активного оператора.

## GetPeriodReport — звіт за діапазон дат

Білдер: `olexml.ReportPeriodXML`. Дати у форматі **DDMMYYYY** (без
роздільників). Frontend надсилає YYYY-MM-DD; `convertDateFormat` переставляє
порядок.

```xml
<InputParameters><Parameters FN="4000167785" StartDate="01122024" EndDate="31122024" /></InputParameters>
```

## Вихід (усі три)

```xml
<OutputParameters>
  <Parameters CL="..." CheckID="..." FN="..."
              L0="line 0" L1="line 1" ... />
</OutputParameters>
```

Парсер: спільний із `GetCheck` — `parseStatusBarXML`. Результат
загортається в `dto.ReportResult` (`Success`, `Lines`, `CheckID`, `FN`,
`Error`).

## Викликачі

- WS-команди: `device.command.report-x`, `device.command.report-z`,
  `device.command.report-period` → `CommandHandler.HandleReportX/Z/Period`.
- Frontend: `DeviceService.reportX/reportZ/reportPeriod`. UI: кнопки звітів
  у заголовку сторінки чеків (X-звіт видимий, доки зміна відкрита, Z-звіт за
  діалогом підтвердження).
