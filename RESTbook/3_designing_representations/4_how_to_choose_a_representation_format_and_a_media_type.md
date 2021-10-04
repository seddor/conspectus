# How to Choose a Representation Format and a Media Type

Можно использовать множество типов данных, более подходящих для ситуации.

Обычно для для передставлений используют xml или json.

При использовании изображений, pdf или подобного можно добавить заголовок **Content-Disposition**:
```
Content-Disposition: attachment; filename=<status.xls>
```
для того, чтобы сообщить клиенту, что данный ответ стоит сохранить в ФС.

[Все стандартизированные форматы](https://www.iana.org/assignments/media-types/media-types.xhtml)
