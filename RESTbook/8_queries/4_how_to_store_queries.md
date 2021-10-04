# How to store queries

При использовании POST для запросов(8.3) возникает проблема кеширования, т.к. POST обычно не кешируется. 

Для подобных случаев кеширование стоит реализовывать так:

* Клиент делает POST запрос
* На основе запроса создаётся новый ресурс, который будет доступен по GET.
* Клиенту возвращается код ответа **201 (Created)** и заголовок **Location** со ссылкой на созданый ресурс.

```
# Request
POST /jobs HTTP/1.1
Host: www.example.org
Content-Type: application/x-www-form-urlencoded
keywords=web,ajax,php&amp;industry=software&amp;experience=5&amp;...
# Response
HTTP/1.1 201 Created
Content-Type: application/xml;charset=UTF-8
Location: http://www.example.org/query/1
Content-Length: 0
```

Далее клиент может запросить ответ:

```
# Request
GET /query/1 HTTP/1.1
Host: www.example.org
# Response
HTTP/1.1 200 OK
Content-Type: application/xml;charset=UTF-8
Date: Wed, 28 Oct 2009 07:22:34 GMT
Cache-Control: max-age:3600
Expires: Wed, 28 Oct 2009 08:22:34 GMT
<postings xmlns:atom="http://www.w3.org/2005/Atom" xml:base="http://www.example.org">
	<posting>
		<atom:link rel="self" href="/job/499"/>
		...
	</posting>
	<posting>
		<atom:link rel="self" href="/job/863"/>
		...
	</posting>
	...
</postings>
```

