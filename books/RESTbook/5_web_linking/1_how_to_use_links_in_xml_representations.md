# How to use links in XML representations

Для ссылок в xml можно использовать `atom:link`, он схож с ссылками в HTML:
```xml
<atom:link link href="http://east-nj1.photos.example.org/987/nj1-1234"
    type="image/jpeg"
    rel="alternate"
    title="Sunset view from our backyard"/>
```
* href — URI
* rel — тип ссылки
* title (опционально) — заголовок ссылки
* type (опционально) — хинт для медиа тип контента возвращаемого по ссылке
* hreflang (опционально) — хинт для языка представления контента по ссылке
* length (опционально) — хинт для размера контента по ссылке

Так же возможно и использования просто прямых URI:
```xml
<user>
    <uri>http://www.example.org/user/001</uri>
    <address>http://www.example.org/user/001/address/001</address>
</user>
```
