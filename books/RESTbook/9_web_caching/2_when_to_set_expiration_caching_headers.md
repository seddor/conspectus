# When to Set Expiration Caching Headers

Кеш нужно использовать для  успешных ответов **GET** и **HEAD** запросов.

Так-же кеш можно добавить для следующих ответов

* 300 (Multiple Choices)
* 301 (Moved Permanently)
* 400 (Bad Request)
* 403 (Forbidden)
* 404 (Not Found)
* 405 (Method Not Allowed)
* 410 (Gone)

POST не должен кешироваться.

