Здравствуйте!

https://docs.google.com/spreadsheets/d/1B3phU8RTlo5K9eRrdSe0JCHPRMo6x_A_rH3aiLB4vs8/edit?usp=sharing

Ниже, комментарии к оценкам.

# Agility

## Монолит

Оценка: 1

Сильно зависит от размера, чем монолит больше тем хуже у него с гибкостью. На первых этапах монолит легко менять, однако, по мере роста это становится делать сложнее, особо если компоненты сильно связаны между собой. для внесения каких-то мзменений требуется большой уровень понимая текущей системы.

## Service-Oriented

Оценка: 3

Большая гибкость на уровне отделительных сервисов, однако сложная система оркестрации.

## Service-Based

Оценка: 4

Большая гибкость сервисов, уровень API не так сложен в понимании как в SOA

## Space-Based

Оценка: 5

Обладает высокой гибкостью, из-за слабой связанности компонентов.

## Event-Driven

Оценка: 4

Высокая гибкость, изменения одних обработчиков сообщений не затрагивает другие, но может вызвать необходимо дорабатывать брокер/медиатор сообщений

## Microservices

Оценка: 5

Компоненты практически не связаны друг с другом, общаются только через внешнее API, легко можно изменять отдельные копоненты.

## Модульный монолит

Оценка: 4

Легко расширять, если код хорошо структурирован по модулям и оно сильно друг от друга не зависят, могут возникнуть проблемы при изменении общих модулей, которые используются во всей системе.

## microkernel

Оценка: 5

Легко расширять, как правило, модули изолированы и общаются между собой посредством микроядра. По идее изменения в одних модулях не должны затрагивать другие.

# Cost

## Монолит

Оценка: 4

Невысокая цена разработки.
Высокая цена поддержки.
Не требует сложной инфраструктуры.

## Service-Oriented

Оценка: 2

Высокая цена разработки.
Высокая цена поддержки из-за сложной оркестрации.
Требует сложной инфраструктуры.

## Service-Based

Оценка: 4

Низкая цена разработки.
Низкая цена поддержки.
Не требует сложной инфраструктуры.
Годится только для не очень больших систем.

## Space-Based

Оценка: 2

Высокая цена разработки.
Высокая цена поддержки.
Требует сложную инфраструктуру.

## Event-Driven

Оценка: 4

Высокая цена разработки.
Средняя цена поддержки.
Не требует сложную инфраструктуру.

## Microservices

Оценка: 3

Средняя цена разработки (в основном легко делать отдельные модули, сложно проектировать систему в целом).
Средняя цена поддержки.
Требует сложную инфраструктуру.

## Модульный монолит

Оценка: 4

Средняя цена разработки (легко делать отдельные модули, нет ограничений на связанность модулей как в микросервисах, но модули нужно нужно правильно проектировать).
Средняя цена поддержки (из-за структурированности проще разобраться в коде, не сложно вонсить изменения).
Не требует сложную инфраструктуру.

## microkernel

Оценка: 3

Высокая цена разработки (система должна быть хорошо спроектирована).
Низкая цена поддержки (если система хорошо спроектирована).
Не требует сложную инфраструктуру.

# Performance

## Монолит

Оценка: 2

Сильно зависит от размера, чем больше тем хуже производительность.

## Service-Oriented

Оценка: 4

Хорошая производительность, но могут быть проблемы из-за сложной оркестрации

## Service-Based

Оценка: 4

Хорошая производительность, могут быть проблемы если используются общие БД (локи и т.п.) и общего API.

## Space-Based

Оценка: 5

Высокая производительность, из-за активного использование кэша.

## Event-Driven

Оценка: 5

Высокая проиводительность из-за асинхронности и возможности параллельной обработки сообщений

## Microservices

Оценка: 4

Высокая производительность отдельных компонентов, однако общение между компонентами накладывает дополнительные ограничения производительности (на получение, десирализацию, валидацию и т.п.)

## Модульный монолит

Оценка: 2

Производительность примерно как у монолита.

## microkernel

Оценка: 1

Производительность примерно как у монолита + дополнительные затраты из-за общение посредством микроядра.

# Scalability

## Монолит

Оценка: 2

В целом монолит сложно масштабировать из-за его размера, можно запустить несколько экземпляров, на разных серверах и сделать балансировку, однако размеры монолита будут накладывать большие ограничения.

## Service-Oriented

Оценка: 4

Можно увеличивать производительность отдельных сервисов, однако будет упор в оркестрацию.

## Service-Based

Оценка: 4

Можно увеличивать количество сервисов, но могут быть проблемы из-за общих уровней, в которые упирается использование этого стиля.

## Space-Based

Оценка: 5

Легко добавлять новые экземпляры, при этом нет узких мест типа общей базы.

## Event-Driven

Оценка: 4

Можно масштабировать каждый обработчик сообщений в отдельности независимо от других, возможны узкие места при оркестрации сообщений.

## Microservices

Оценка: 5

Каждый сервис можно масштабировать отдельно, при этом у них нет узких мест из-за слабой связаности.

## Модульный монолит

Оценка: 3

Возможности такие-же как у монолита, при необходимости можно отключать какие-то модули, что может увеличить производительность.

## microkernel

Оценка: 3

Возможности такие-же как у монолита, при необходимости можно отключать какие-то модули, что может увеличить производительность.

# Testability

## Монолит

Оценка: 3

Зависит от размера, изначально тестировать легко, затем всё сложнее из-за сильной связанности.

## Service-Oriented

Оценка: 4

Легко тестировать отдельные сервисы и систему в целом, сложно уровень оркестрации.

## Service-Based

Оценка: 5

Компоненты разделены и общее API не такое сложное как у SOA, в целом систему не сложно тестировать.

## Space-Based

Оценка: 4

Не сложно тестировать работу системы и отдельных сервисов, могут быть сложности с нагрузочными тестами, т.к. потребуются большие мощности для обеспечения нагрузки.

## Event-Driven

Оценка: 4

Легко тестировать отдельные сервисы, сложно систему в целом, требуется учитывать асинхронность системы.

## Microservices

Оценка: 5

Легко тестировать систему (из-за слабой связанности) и отдельные компоненты.

## Модульный монолит

Оценка: 4

Проще тестировать чем монолит из-за большей стуктурированности что позволяет тестировать отдельные модули, однако могут быть проблемы из-за связанности.

## microkernel

Оценка: 5

Можно тестировать отдельные модули и функции микроядра, в целом система малосвязана и не должна требовать сложный тестов целой системы.
