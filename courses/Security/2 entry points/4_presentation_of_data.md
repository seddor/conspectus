# Представление данных

* Мапинг урла в параметры
Например: http://site.com/catalog/product/id/123
* Base64
Например: http://site.com/?in=QTM4O0IxNjtGOTk7SzlxCg%3D%3D -> QTM4O0IxNjtGOTk7SzlxCg== -> A38;B16;F99;K21
%3D — uri encoded =
Base64 можно кодировать в консоли:

Кодирование: `echo 'Foo' | base64`
Декодирование: `echo 'Rm9vCg=='|base64 -d`