# How to Interpret Entity Headers

Заголовки запроса:
* **Content-Type** — При получении запроса без этого заголовка слать **400 Bad Request**. 
* **Content-Encoding** — обработать запрос в соответстваии с описанием из заголовка.
* **Content-Language** — использовать для языка представления.

