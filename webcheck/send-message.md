# SendMessage

Один метод, два режими, що обираються через `messagetype`: надіслати
посилання на чек через Viber/SMS або запитати залишок балансу повідомлень
пристрою.

Двокрокова схема. Білдер: `olexml.SendMessageXML`.

## Значення messagetype

| Значення | Сенс |
|-------|---------|
| `1`   | Лише Viber |
| `2`   | Лише SMS |
| `3`   | Гібридний — спершу Viber, у разі невдачі SMS |
| `4`   | Запит балансу (повідомлення не надсилається) |

Ми використовуємо `3` для надсилання та `4` для балансу.

## Вхід

Запит балансу (`messagetype=4`):

```xml
<InputParameters><Parameters FN="4000167785" CheckID="T8ru6rJYmJo" messagetype="4"/></InputParameters>
```

Надсилання (`messagetype=3`):

```xml
<InputParameters><Parameters FN="4000167785" CheckID="T8ru6rJYmJo" messagetype="3" Dest="380XXXXXXXXX"/></InputParameters>
```

| Атрибут       | Тип  | Значення |
|---------------|------|---------|
| `FN`          | int  | Фіскальний номер пристрою |
| `CheckID`     | str  | Фіскальний номер чека. **Те саме значення, що й `TaxNum`** у [`GetCheckCloudurl`](get-check-cloudurl.md), але назва атрибута інша згідно з драйвером. |
| `messagetype` | int  | Див. таблицю вище |
| `Dest`        | str  | Телефон отримувача, лише цифри (`380XXXXXXXXX`). Обов'язковий для `messagetype` 1/2/3. **Пропустіть для `4`** — передача порожнього `Dest=""` спричиняє `Err=1004` "Неверный формат XML". |

## Вихід

Успіх:

```xml
<OutputParameters>
  <Parameters Err="0" FN="4000167785" Remainder ="50"/>
</OutputParameters>
```

Помилка:

```xml
<OutputParameters>
  <Parameters Err="91" ErrHelp="Не вірно вказаний номер телефону" version="6.0.7.1338"/>
</OutputParameters>
```

| Атрибут     | Значення |
|-------------|---------|
| `Err`       | `"0"` = успіх; ненульове значення = код помилки |
| `Remainder` | Кількість повідомлень, що залишилися (зменшується після успішного надсилання) |
| `ErrHelp`   | Опис помилки кирилицею |

> ⚠️ Драйвер іноді видає зайвий пробіл перед `=`, наприклад,
> `Remainder ="50"`. Валідний XML, але збиває наївні регулярні вирази
> `Remainder="..."`. Наші парсери використовують `\s*=\s*`. Див.
> [особливості](quirks.md).

Парсери: `parseMessageBalanceStatusXML`, `parseSendMessageStatusXML` у
`internal/service/ole_windows.go`.

## Викликачі

- **Баланс**: `OLEService.GetMessageBalance` → WS-обробник
  `HandleMessageBalanceQuery` (`device.query.message-balance`) → frontend
  `DeviceService.getMessageBalance`. Спрацьовує, коли користувач відкриває
  панель повідомлень.
- **Надсилання**: `OLEService.SendMessage` → WS-обробник `HandleSendMessage`
  (`device.command.send-message`) → frontend
  `DeviceService.sendCheckMessage`. Спрацьовує кнопкою "Надіслати" після
  того, як користувач вводить телефон.

Frontend видаляє пробіли / дефіси / дужки з вводу користувача, потім backend
видаляє провідний `+` перед передачею як `Dest`.
