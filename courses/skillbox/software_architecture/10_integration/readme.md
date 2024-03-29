# Интеграция

## Общие аспекты. Синхронность и асинхронность

Интеграция — это объединение отдельных составных частей с помощью определённых действий в единое целое, либо их встраивание в уже существующий целостный объект.

### Синхронное взаимодействие

Случаи, когда время ответа важно:

* Работа со счетами
* Показания датчиков
* Работа с динамическими данными и т.д.

**TimeOut** — время, в рамках которого ответ от системы для нас актуален.

#### Выбор TimeOut'a

Необходимо понять:

* как часто происходит изменения с объектом
* как много будет TimeOut'ов при выбранном значении
* комфортно ли пользователю

### Асинхронное взаимодействие

Случаи, когда время ответа **не** важно:

* выполнение тяжелых задач
* большие вычисления
* работа со статическими данными и т.д.

**Асинхронное ≠ реактивное**

Нужно помнить о следующем:

* опрос окончания обработки
* обработка ошибок

## Классификация интеграций

При проработке интеграции нужно ответить на несколько вопросов:

* Какую проблему решает?
* Можно ли обойтись без неё?
* Увеличит ли она связанность системы?
* Сколько стоит внесение изменений?
* Как сложно её дорабатывать?

### Доработка системы

При дорабатывали системы для интеграции:

* Доработка не должна привести к сильной связанности в приложении
* Необходимо помнить, что доработка, сделанная в рамках интеграции сервисов, будет потом переиспользована для интеграции каких-то других клиентов. Это хорошо бы учитывать и следовать подходу API first, когда API по сути является продуктом, сделанный для внешних клиентов.
* Хорошо понять и оценить, возможно ли не делать эту интеграцию, и возможно ли передать эту интеграцию на какой-то другой уровень.

### Выбор технологии

При выборе нужно обратить внимание на:

* Можно ли использовать готовое решение
* Не станет ли выбор технологии вендор-локом
* Соответствует ли технология стратегии развития
* Соответствует ли требованиям безопасности
* Стоимость внедрения технологии
* Является ли технология мультиязычной и возможно ли её использовать на разных платформах

### Данные

Необходимо подумать о данных, которые будут передаваться между сервисами.

- **Формат данных**
  - Возможность использовать один формат данных
  - Необходимость разработки промежуточного слоя (трансляторы, медиаторы, ESB)
- **Своевременность данных**
  - Актуальность данных
  - Требования к TimeOut'ам
  - Пакетная передача
  - Уведомления о новых данных
  - Постоянное обращение к источнику данных

### Синхронное или асинхронное взаимодействие

* **Синхронное взаимодействие:**
  * простота разработки
  * простота поддержки
  * наглядность работы
  * возможность управлять ожиданием
* **Асинхронное взаимодействие:**
  * выигрыш в производительности
  * нет затраты ресурсов на ожидание ответа
  * усложнение интеграции
  * сложная поддержка

### Надёжность

* Требование к повторению запросов
* Требование к обработке ошибок
* Требование к восстановлению запроса
* Логирование запросов
* Использование программных и аппаратных способов обнаружения ошибок

### Выбор способа интеграции

* Передача файлов
* Интеграция через БД
* Удалённый вызов системы
* Обмен сообщениями
* Потоковая обработка

### Передача данных

Для передачи данных можно использовать различные протоколы, например:

* FTP
* SFTP
* AFP
* BitTorrent
* S3 и т.д.

Также можно комбинировать различные протоколы

### Интеграция через БД

Если нужно сделать интеграцию через БД, то необходимо:

* Продумать общие таблицы
* Подготовить специальные вью (view) для каждого из сервисов
* Важно помнить, что данное решение очень сложно масштабировать

### Удалённый вызов системы

Есть способ интеграции через удалённый вызов системы, посредством:

* HTTP
* SOAP
* GraphQL
* RPC и т.д.

Главное понимать, что в рамках данной интеграции нужно запрашивать какой-то сервис и ждать ответ, при этом можно построить и асинхронное взаимодействие, когда сервис удалённо просят выполнить какую-то задачу, а потом запрашивается статус этой задачи.

###  Обмен сообщениями

Можно использовать сообщения для интеграции, с помощью:

* IBM MQ
* Kafka
* RabbitMQ
* ActiveMQ
* MQTT
* RSS и т.д.

Обмен сообщениями происходит так:

- Обычно сервис кладёт в какую-то очередь сообщения, из которой клиент читает
- Обычно используется FIFO (First in, first out)
- В рамках этого взаимодействия сообщения могут вычитывать как по одному, так и пакетами
- Также можно поставить ещё очередь, которая будет получать сообщения от клиента и отправлять их в сторону сервиса, для реализации синхронного взаимодействия и связи этих сообщений через ID, которые будет передаваться между сервисами
- Обмен сообщениями это чаще всего асинхронный паттерн, но само вычитывание сообщений происходит синхронно, при этом если сервис не смог прочитать сообщение, он возвращает его обратно в очередь и пробует ещё раз
- Если возникает сообщение, которое бесконечно ломает сервис, то можно использовать паттерн DLQ (dead letter queue), для этих сообщений создаётся отдельная очередь, куда помещаются сообщения ломающие сервис, при этом стоит помнить, что это ошибки не бизнес-логики, а ошибки, которые появились в момент парсинга сообщения, десериализаци/сериализации его.
- Для сообщений у которых возникли проблемы при работе с БД хорошо бы придумать тайм-аут, чтобы дать БД некоторое время отдыха, или сделать отдельную очередь для таких сообщений, чтение из которой будет происходить не сразу по поступлению, а с неким интервалом.

Также есть реализация через топики, где вместо очереди используется топик в которые пишутся некие транзакционные события, а клиенты могут их считывать. При этом события в этом топике остаются, а меняется лишь сдвиг, который записывается для каждого клиента. Этот подход позволяет всегда хранить сообщения на диске, он очень подходит для SQRS, когда необходимо, иногда, перечитывать все события, которые появлялись в топике. Топик можно использовать как хранилище событий и настроить как долго они там будут храниться. Это можно быть количество сообщений, время или какой-то объём, который они занимают.

Необходимо помнить, что все эти взаимодействия намного сложнее синхронных.

### Потоковая передача

Для интеграции может быть использована потоковая передача данных. Для передачи, аудио, видео, стриминга. Протоколы:

* RTSP
* RTMP
* SRT и т.д.

## Правила построения интеграций. Отказоустойчивость

### Микросервисы

Микросервисы это всегда интеграции, а т.к. они являются мелкими частями системы, то и интеграций тоже большое количество. Таким образом может появится какой-то отказ, затем отказывают сервисы связанные с ним, затем сервисы связанные с ними и т.д. пока не откажет вся система. Чем больше интеграций, тем больше шанс зарождения такого отказа.

Для отлавливания таких отказов необходимо сделать дополнительную работу с ошибками, которые могли бы препятствовать таким отказам.

### Time out

Допустим есть тайм аут в 30 сек. При этом есть нагрузка в 5 запросов в секунду.

Если сервис начнёт уходить в тайм аут: то количество запросов будет расти, потому что клиент до сих пор ждёт предыдущего ответа на запрос, в какой-то момент количество коннектов, которое устанавливается между клиентом и сервисом достигнет максимума и в итоге получится загрузка и клиента, и сервиса, и, скорее всего, отказ.

Чтобы этого избегать нужно грамотно побирать тайм ауты, чтобы за то время, когда ожидается тайм аут, не произошло, что были исчерпаны все ресурсы, которые необходимо для обработки каких-либо запросов или ответов.

Также важно быстро отвечать ошибкой, если есть понимание, что в сервисе что-то пошло не так.

### Circle breaker

Есть паттернг Circle breaker, который позволяет разорвать круг с постоянными ошибками.

По сути это некий модуль, встраиваемый между клиентом и сервисом и отвечающий за маршрутизацию запросов, которые генерит клиент. Circle breaker обычно знает о всех инстансах сервиса и распределяет запросы между ними.

Он работает так: допустим какой-то сервис вернул ошибку, он это анализирует, понимает что сейчас пришёл таймаут или 500я ошибка, ждёт какого-то порога, который настроен (например, 5 ошибок в минуту, зависит от требований системы), и после этого он переключает все запросы на другой сервис.

У circle breaker есть 3 статуса для сервисов:

* Closed —когда сервис начал возвращать ошибки и нужно перевести его запросы на другие сервисы
* Half open — возникает через некоторое время, после возникновения ошибок, и Circle breaker начинает иногда посылать в него запросы, для проверки, восстановился ли сервис. Если определённое количество запросов нормально прошло, то сервис переводится в Open
* Open — сервис открыт и на него идут запросы

### Retry

Паттерн по сути повторяет запрос, который вернулся с ошибкой. Тут важно выставить:

* Время между попытками
* Число попыток

### Fallback

Альтернативный сценарий работы сервиса после повторных запросов.

Это должен быть самый простой функционал, который точно всегда сработает.

### Обмен сообщениями

Повышает отказоустойчивость системы из-за того, что сами очереди сообщений являются неким буфером, которые позволяют беферизировать сообщения, при высокой нагрузке от клиентов, при этом сервис будет так же спокойно их вычитывать и обрабатывать. При этом нужно учитывать все случаи, когда какие-то сервисы не могут прочитать сообщения, или БД перестала работать и т.п.

Очереди позволяют легко масштабироваться.

### Cron jobs

Это не паттерн, тут надо обращать внимание при их настройке.

Обычно cron jon'ы настраивают какими-то кратными числами (каждые 2 часа, каждые 5 часов и т.д). Однако, велика вероятность, что таким образом в одно время будет запускаться слишком много джоб. Поэтому при настройке нужно следить, чтобы выбиралось время не кратное друг другу.

Или можно сделать, чтобы задачи выполнялись не каждые два часа а два часа от времени, когда была завершена прошлая эта задача, таким образом к времени запуска всегда будет добавляться время на выполнение джобы, таким образом вероятность, чтобы джобы пересекаться сокращается.
