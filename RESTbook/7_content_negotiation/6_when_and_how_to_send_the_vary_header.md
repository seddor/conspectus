# When and how send the vary header

В заголовок **Vary** влкючается заголовки, разделённые запятой, которые были применены к ответу.

```
# Request for English representation
GET /status HTTP/1.1
Host: www.example.org
Accept-Language: en;q=1.0,*/*;q=0.0
# Response
HTTP/1.1 200 OK
Content-Language: en
Vary: Accept-Language
```

