# GetCheckCloudurl

Повертає авторитетне посилання сервера WebCheck на чек (PDF на
`che.ck.ua`). Замінює попередній URL `cabinet.tax.gov.ua`, що складався на
боці frontend — cloud URL є канонічним, і користувач може його скопіювати.

Двокрокова схема. Білдер: `olexml.GetCheckCloudurlXML`.

## Вхід

```xml
<InputParameters><Parameters FN="4000167785" TaxNum="T8ru6rJYmJo" type="0"/></InputParameters>
```

| Атрибут   | Тип  | Значення |
|-----------|------|---------|
| `FN`      | int  | Фіскальний номер пристрою |
| `TaxNum`  | str  | Фіскальний номер чека |
| `type`    | int  | Тип документа. Згідно з документацією постачальника обов'язковими є лише `FN` + `TaxNum`, але ми передаємо `type` для узгодженості форми з `GetCheck`. Драйвер толерує додатковий атрибут. |

> ⚠️ Те саме значення тут називається `TaxNum`, а в
> [`SendMessage`](send-message.md) — `CheckID` — драйвер очікує різні назви
> атрибутів для кожного методу, вони **не** взаємозамінні.

## Вихід

Успіх:

```xml
<OutputParameters>
  <Parameters Err="0" FN="4000167785" URL="https://che.ck.ua/T8ru6rJYmJo.pdf"/>
</OutputParameters>
```

Помилка (наприклад, чек не знайдено):

```xml
<OutputParameters>
  <Parameters Err="Чек не найден" ErrHelp="Чек не найден" version="3.2.5.0169"/>
</OutputParameters>
```

| Атрибут   | Значення |
|-----------|---------|
| `Err`     | `"0"` = успіх; ненульове значення = код помилки |
| `URL`     | Cloud URL у разі успіху |
| `ErrHelp` | Зрозумілий людині опис помилки кирилицею |

Парсер: `parseCheckCloudURLStatusXML` у `internal/service/ole_windows.go`.

## Викликач

`OLEService.GetCheckCloudURL` → WS-обробник `HandleCheckCloudURLQuery`
(`device.query.check-cloud-url`) → frontend
`DeviceService.getCheckCloudUrl`. UI: панель cloud-URL у
`features/checks` (кнопка "Лінк ПРРО").
