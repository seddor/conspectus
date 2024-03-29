# How to implement character encoding negotation

* По умолчанию нужно использовать **UTF-8**, и отправлять ответ в кодировке по умолчанию, если клиент не прислал заголовок **Accept-Charset**
* Если **Accept-Charset** есть, то нужно его распарсить, отсортировать по убыванию **q**, и отправить ответ с первой, поддерживаемой сервером кодировкой. 
* Если сервер не поддерживает кодировки из списка и **Accept-Charset** не содержит **\*; q=0.0**, то прислать ответ в **UTF-8**.

Во всех случаях, если медиа тип текстовый и поддеживает параметр **charset**, **charset** нужно вклоючить в заголовок **Content-Type**. Также нужно включить заголовок **Vary** (7.6).
