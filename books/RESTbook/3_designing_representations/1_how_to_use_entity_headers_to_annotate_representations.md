# How to use entity headers to annotate representations

Для ответов стоит использовать такие заголовки:
* **Content-Type** — тип(MIME) ответа
* **Content-Length** — размер ответа в байтах, в HTTP/1.1 можно использовать **Transfer-Encoding: chunked**
* **Content-Language** — язык ответа
* **Content-MD5** — для проверки целдостности, если необходмо
* **Content-Encoding** — когда теле ответа кодируется, например используя **gzip**, **compress** или **defalte** кодирование
* **Last-Modified** — для обозначения когда в последний раз ресурс изменялся сервером
