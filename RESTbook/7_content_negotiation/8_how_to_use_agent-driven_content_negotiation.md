# How to Use Agent-Driven Content Negotiation

Agent-Driven Content Negotiation полезны когда клиент не может использовать **Accept-***  заголовки. Для таких случаев нужно предоставить разные URI для каждого представления.

## Способы разделения:

* Параметры запроса — можно использовать для разделения по медиа типам:

  ```
  http://www.example.org/status?format=json
  http://www.example.org/status?format=xml
  http://www.example.org/status?format=csv
  ```

  

* URI extension — можно дописывать требуемый формат после точки в запросе:

  * status.atom для application/atom+xml
  * status.json для application/json
* поддомены — можно использовать для разделения по языкам
  *  en.wikipedia.org — аннлийский
  *  de.wikipedia.org — немецкий

В ответ можно также добавить alternate ссылки на альтернативные представления
