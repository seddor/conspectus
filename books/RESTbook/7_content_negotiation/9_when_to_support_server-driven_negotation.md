# When to support Server-Driven Negotiation

Нужно поддерживать несколько вариантов представления только когда они необходимы клиентам или все варианты контента содержат одинаковую информацию, если она различается то нужно использовать разные URI для каждого варианта.

Перед тем как реализовывать управление контентом по средством сервера(с помощью заголовков **Accept-***) надо учитывать случаи когда это невозможно или затруднительно:

* Флоу приложения может отличатся для разных типов представлений, Например, у клиентов требующих HTML-представления часто флоу может различатся от флоу для, например, XML-представлений.
* Однозначно интерпретировать нужный вариант ответа на основе **Accept** заголовка с несколькими типами данных и разными **q** не тривиально, некоторые фреймворки могут этого не поддерживать.
* Для глобальных сервисов для разных регионов могут быть разные правовые и бизнес требования, в таких случаях лучше разделять эти ресурсы с помощью Agent-Driven Negotiation(7.8).
* Кеш может не правильно работать или иметь ограничения на варианты представления.