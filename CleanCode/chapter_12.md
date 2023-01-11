# Формирование архитектуры

## Четыре правила

Архитектуры можно считать "простой" если она:

* обеспечивает прохождение всех тестов;
* не содержит дублирования кода;
* выражает намерения программиста;
* использует минимальное количество классов и методов.

### Правило №1: выполнение всех тестов

Прежде всего система должна делать то, что задумывал проектировщик. Если нет простого способа убедиться в том, что она действительно решает свои задачи, то результат выглядит сомнительно. в

Системе прошедшая все тесты контролируема.

### Правило №2-4: переработка кода

#### Отсутвие дублирования

Дублирование — главный враг хорошо спроектированной системы. Егго последствия — лишняя работа, лишний риск и лишняя избыточная сложность.

#### Выразительность

Код должен чётко выражать намерения своего автора.

Хороший выбор имён помогает выразить намерение. Имя класса или функции должно восприниматься "на слух", а когда читатель разбирается в том, что делает класс, это не должно вызывать у него удивление.

Относительно небольшой размер функций и классов также помогает в выразительности.

Использование общепринятых паттернов и названий упрощает понимание кода.

Зорошо написанные тесты тоже выразительны. Их можно рассматривать как разновидность документации, построенной на конкретных примерах. Читая код тестов у разработчика должно сложиться хотя бы общеее представление о том, что делает класс.

#### Минимум классов и методов

При устрании дублирования, улучшения выразительности кода и реализации принципа единой ответственности, главное не зайти слишком далеко и не создать слишком много крошечных клссов и методов.

Наша цель — чтобы система была компактной, но при этом одновременно сохранить компактность функций и классов. Однако стоит помнить, что это правила простой архитектуры имеет наименьший приоритет, поэтому стемясь свести к миниму клоичество классов и функций важно в превую очередь заботиться о исполнеии остальных правил.