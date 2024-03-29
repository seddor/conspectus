1.

| **Описание API/системы**                                     | **Возможные векторы атак**                                   | **Способы защиты от векторов атак**                          |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Внешний сервис заказов — сервис в который отправляются заказы после создания на сайте | 1. Инъекция — в данные заказа можно строить вредоносный код. | 1. Валидация и экранирование приходящих данных.              |
|                                                              | 2. Недостатки аутентификации, если между внутренними сервисами системы нет аутентификации | 2. Установка аутентификации между сервисами                  |
|                                                              | 3. Незащищённость конфиденциальных данных — по сети передаются все данные заказа, которые могут утечь | 3. Шифрование передаваемых данных                            |
|                                                              | 4. Небезопасная десериализация, внешние сущности XML — в запрос на создание заказа можно встроить вредоносный код | 4. Использование JSON для передачи данных заказа в запросе к нашей системе. |
|                                                              | 5. Подделка запросов со стороны сервера (SSRF)               | 5. Аналогично 1.                                             |
| Внешний сервис местоположения — сервис используемый для получения местоположения клиента | 1. Инъекция — в параметры запроса можно строить вредоносный код. | 1. Валидация и экранирование приходящих, в нашем случае передаются координаты, поэтому допустимы только числа с плавающей запятой |
|                                                              | 2. Недостатки аутентификации, если между внутренними сервисами системы нет аутентификации | 2. Установка аутентификации между сервисами                  |
|                                                              | 3. Небезопасная десериализация, внешние сущности XML — в запрос на получения местоположения можно встроить вредоносный код | 3. Аналогично 1.                                             |

2. Схемы взаимодействия для сценариев взаимодействия с внешними сервисами (создание заказа и получение местоположения), красным цветом обозначены моменты связанные с безопасностью.

   > Какую аутентификацию собираетесь использовать между сервисами?

Можно использовать аутентификацию с помощью сертификатов и/или цифровой подписи.

> Проанализировать систему и понять, какие первые шаги нужно сделать для  обеспечения безопасности пользовательских данных и почему. Аргументы  стройте на времени, которое уйдёт на доработку, и пользе, которую она  принесёт.

Для начала можно или убрать возможно получать информацию о заказе только по номеру, без авторизации клиента на сайте, или, если это делать нельзя, то добавить данные которые нужно ввести пользователю, например, добавить email, чтобы данные заказа можно было получить только зная номер заказа и email на который он был оформлен, это должно уменьшить шанс утечки данных с этой страницы.

Дополнительно, можно добавить на этой страницы ограничения на количество запросов с одного IP, с одним email, чтобы исключить варианты перебора номеров заказа с утекшими email.

Далее нужно уже дорабатывать взаимодействие с внешней системой и созданием заказа, чтобы исключить возможность утечки на этих этапах.

> Проработать систему управления заказом, чтобы злоумышленники не могли похитить данные пользователей.

Описал в таблице и на схеме, в частности предлагается:

1. Добавить валидацию и экспейпинг данных
2. Авторизацию между сервисами
3. Шифрование данных при передаче

> >Рассмотреть и защититься от следующих векторов атак:
>
> 1. Injections (инъекции);

Валидация и эскейпинг входящих данных.

> 1. Broken Authentication (нарушенная аутентификация);

Авторизация между сервисами

> 1. Sensitive Data Exposure (незащищённость конфиденциальных данных);

Шифрование данных

> 1. XML External Entities (XXE) Insecure Deserialization (внешние сущности XML, небезопасная десериализация);

Использование JSON, отключение/избегания небезопасных функций библиотек для парсинга.

Выделение демилитаризованной зоны, мне кажется, что в нашем случае будет малоэффективно, т.к. почти все сервисы у нас общаются с клиентом напрямую (каталог отдаёт информацию о товарах, сервис заказов создаёт заказы, сервис клиентов отвечает за аккаунт клиента, сервис цен напрямую с клиентом не общается, но он не содержит каких-то критичных данных, чтобы его так защищать).

> 1. Broken Access Control (нарушение контроля доступа).

Правильная настройка прав доступа. Возможно, какой-то монтиоринг, чтобы отслеживать подозрительное поведение пользователей, например запрос файлов, к которым у пользователя нет доступа.