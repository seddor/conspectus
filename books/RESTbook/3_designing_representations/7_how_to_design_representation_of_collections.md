# How to Design Representations of Collections

* Коллекция должна включать self link
* Если коллекция содержит страницы, то нужны сслыки на следующую/предыдущую страницы(если они имеются).
* Информацию о размере коллекции.
```
# Request
GET /articles?contains=cycling&start=10 HTTP/1.1
Host: www.example.org

# Response
HTTP/1.1 200 OK
Content-Type: application/xml;charset=UTF-8
Content-Language: en
```
```xml
<articles total="1921" xmlns:atom="http://www.w3.org/2005/Atom">
    <atom:link rel="self" href="http://www.example.org/articles?contains=cycling&start=10"/>
    <atom:link rel="prev" href="http://www.example.org/books?contains=cycling"/>
    <atom:link rel="next" href="http://www.example.org/books?contains=cycling&start=20"/>
<article>
    <atom:link rel="self" href="http://www.nytimes.com/2009/07/15/sports/cycling/15tour.html"/>
    <title>For Italian, Yellow Jersey Is Fun While It Lasts</title>
    <body>...</body>
</article>
<article>
    <atom:link rel="alternate" href="http://www.nytimes.com/2009/07/27/sports/cycling/27tour.html"/>
    <title>Contador Wins, but Armstrong Has Other Victory</title>
    <body>...</body>
    </article>
        ...
</articles>
```
