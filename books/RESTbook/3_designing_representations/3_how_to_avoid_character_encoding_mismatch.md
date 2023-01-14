# Hot to Avoid Charecter Encoding Mismath

При отправке предсталения, если медиа тип поддерживает параметр **charset** то стоит отправлять его в ответе:
```
Content-Type: application/xml;charset=UTF-8
```

Для xml не стоит использовать **text/xml**, т.к. для него дефолтной является кодировка us-ascii, в то время как **application/xml** использует UTF-8.
