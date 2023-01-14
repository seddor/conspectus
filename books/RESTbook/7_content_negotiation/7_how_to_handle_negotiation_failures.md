# How to handle Negotiation Failures

Когда сервер не может ответить в формате нужном клиенту и если клиент явно содержит `*/*;q=0.0 ` в **Accept** нужно вернуть **406 (Not Acceptable)** с телом содержащим список возможных представлений или ссылку на документацию со списком.

Если сервер не поддерживает запрашиваемый **Accept-Encoding**, то ответ отправляется без кодирования.

```
# Request
GET /user/001/followers HTTP/1.1
Accept: application/json,*/*;q=0.0
```



```
# Response
406 Not Acceptable
Content-Type: text/html;charset=UTF-8
Link: <http://www.example.org/errors/mediatypes.html>;rel="help"
<html>
	<head>
		<title>JSON Not Supported</title>
	</head>
	<body>
		<p>This server does not support JSON. See <a href="http://www.example.org/errors/mediatypes.html">help</a> for alternatives.</p>
	</body>
</html>
```

