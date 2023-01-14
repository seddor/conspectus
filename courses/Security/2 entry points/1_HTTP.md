# HTTP протокол

## Запрос

```
1 GET https://www.foo.com/ HTTP/1.1
2 Host: www.foo.com
3 User-Agent: Mozilla/5.0 Firefox
4 Accept: text/html
5 Acept-Language: en-US,en;
6 DNT: 1
7 Cookie: B=ergesge
8 Connection: keep-alive
```

1. Содержит метод(в данном случае GET), запрашиваемый ресурс, версия протокола
2. Host присутвует только в HTTP/1.1, форсируется клиентом, является одной из точек входа
3. Определяет устройство клиента, возможно изменять
7. Передаётся при наличии, используется как идентификатор, может быть получено при Set-Cookie в ответе.

## Ответ

```
HTTP/1.1 200 OK
Date: Mon, 20 Apr 2015 09:25:24
Server: Apache
X-Powered-By: PHP/5.2.4-2ubuntu5wm1
Last-Modified: Wed, 11 Feb 2009 11:20:59 GMT
Content-Language: ru
Content-Type: text/html; charset=utf-8
Content-Length: 1234
```

## POST

X-Requested-With: XMLHttpRequest — указаывает посредством чего был создан запрос, в данном случае запрос из JS