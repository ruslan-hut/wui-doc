# GetCheck

Завантажує чек у внутрішній стан COM-об'єкта. Використовується разом із
`StatusBarXML` для відображення попереднього перегляду чека на сторінці
чеків.

Двокрокова схема. Білдер: `olexml.GetCheckXML`.

## Вхід

```xml
<InputParameters><Parameters FN="4000167785" TaxNum="T8ru6rJYmJo" type="0"/></InputParameters>
```

| Атрибут   | Тип  | Значення |
|-----------|------|---------|
| `FN`      | int  | Фіскальний номер пристрою |
| `TaxNum`  | str  | Фіскальний номер чека (колонка БД `taxNum`) |
| `type`    | int  | Тип документа — `0` продаж, `1` повернення тощо. |

## Вихід (після `StatusBarXML`)

```xml
<OutputParameters>
  <Parameters CL="16" CheckID="..." FN="..."
              L0="line 0" L1="line 1" ... L15="line 15"/>
</OutputParameters>
```

| Атрибут   | Значення |
|-----------|---------|
| `CL`      | Загальна кількість рядків |
| `CheckID` | Ідентифікатор чека |
| `FN`      | Відлуння фіскального номера |
| `L0..LN`  | Рядки чека, упорядковані за N |

Парсер: `parseStatusBarXML` у `internal/service/ole_windows.go`.

## Викликач

`OLEService.GetCheckPreview` (кеш у пам'яті, максимум 50 записів, витіснення
за LRU). Frontend: `DeviceService.getCheckPreview` →
WS-повідомлення `device.query.check-preview`.
