# Системы

## Отедление инициализации системы от её использования

В программных системах фаза инициализации, в которой конструируются объекты приложений и склеиваются основные зависимости, должна отделяться от логики времени выполнения.

Фаза инициализации присутвует в каждом приложении. Это первая из областей ответственности (concerns). Разделение ответственности относится к числу важнейших приёмов.

### Отделение main

Один из способов отделения инициализации от использования — перемещения всех аспектов инициализации в main/модули вызываемыме из main (index.php/отдельный класс от всей системы). Остальной код пишется с предположением, что все системы уже инциализированы.

### Внедрение зависимостей (DI)

DI — мощный инструмент отделения инициализации от использования. В контексте управление зависимостями объект не должен брать на себя ответственность за создание экземпляров зависимостей. Он передаёт эту обязаность другому механизму.

Класс не принимает непосредственных действий по разрешению зависимостей, вместо этого он предоставляет set-методы и/или аргументы конструктора, используемые для внедрения зависимостей. В процессе инициализации контейнер DI создаёт необходимые инстансы объектов (обычно по требованию) и использует аргументы set-методов/конструкторов для скрепления зависимостей. Фактически используемые зависимые объекты прописываются в конфигурациях или на программном крове в специальном инициализирующем модуле.

## Масштабирование

Возможность посмтроить правильную систему с первого раза — миф.

Архитектура программных систем **может** развиваться последовательно, если обеспечить правильное разделение ответственности.

## Поперечные области ответственности

Такие области, как сохранение объектов, выходят за рамки ественных границ объектов предметной области. Например, все объекты обычно сохраняются по одной стратегрии, с использованием СУБД.

Для таких областей используетсятся термин "поперечные области ответственности"

Аспекктно-ориентированное программирование (АОП) представляет собой универсальный подход для восстановлению модульности для поперечных областей ответственности. В АОП специальные модульные конструкции (аспекты) определяют, в каких точках системы поведение должно меняться некоторым последовательным образом в соответствии с потребностями определённой области отвественности. Определение определяется на уровне декларативного или программного механизма.