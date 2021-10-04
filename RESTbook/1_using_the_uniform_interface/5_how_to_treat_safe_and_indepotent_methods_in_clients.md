# How to traet safe and idempotent methods in clieints

GET, OPTIONS и HEAD должны использоватся только для операций чтения и не должны оказывать каких-либо сайд эффектов.

В случае сетевых или программных ошибок, повторный запрос GET, PUT, DELETE должен быть с заголовками If-Unmodified-Since и/или If-Match.

