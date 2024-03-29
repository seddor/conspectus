# Временные данные

## Часовые пояса

`getutcdate()` — ворзвращает текующе время в UTC.

MySQL для часового пояса использует две настройки

* @@global.time_zone — глобальный часовой пояс.
* @@session.time_zone — сессионный, можетотличатся для каждого пользователя.

По умолчанию записано значение `SYSTEM`, означающее, что пояс берётся из настроек сервера.

Изменить часовой пояс для сеанса:
```SQL
SET time_zone = 'Europe/Moscow';
```

## Создание временных данных

Временные данные можно создать:
* из столбцов типа `date`, `datetime` или `time`
* выполнением встроеный функций возвращаюих такие же типы данных
* Создание строковых представлений, которые потом преобразует сервер.

### Строковые представления

|Компонент |Описание| Диапазон|
|-|-|-|
|YYYY| Год, включая столетие |от 1000 до 9999|
|MM| Месяц |от 01 (январь) до 12 (декабрь)|
|DD| День |от 01 до 31|
|HH| Час |от 00 до 23|
|HHH| Часы (прошедшие) |от –838 до 838|
|MI| Минута |от 00 до 59|
|SS| Секунда |от 00 до 59|

Для того, чтобы сервер интерпретировал строку как временные данные нужно свести её к одному из форматов:
|Тип |Формат по умолчанию|
|-|-|
|Date| YYYY-MM-DD|
|Datetime| YYYY-MM-DD HH:MI:SS|
|Timestamp| YYYY-MM-DD HH:MI:SS|
|Time| HHH:MI:SS|

Если сервер ожидает временные данные(например если строка соотвующего типа) то он автоматически их преобразует.

Если сервер не ожидает то привести данныее можно вручную функцией `CAST`:
```
SELECT CAST('2005-03-27 15:30:00' AS DATE);
```

### Фнкции создания дат 

`str_to_date()` позволяет преобразовывать строки в дату:
```SQL
UPDATE individual
SET birth_date = STR_TO_DATE('March 27, 2005', '%M %d, %Y')
WHERE cust_id = 9999;
```

Компоненты даты:
|Компонент форматирования| Описание|
|-|-|
|%M| Название месяца (от January до December)|
|%m| Номер месяца (от 01 до 12)|
|%d| Число (от 01 до 31)|
|%j| День года (от 001 до 366)|
|%W| Дни недели (от Sunday до Saturday)|
|%Y| Год, четырехзначное число|
|%y| Год, двузначное число|
|%H| Час (от 00 до 23)|
|%h| Час (от 01 до 12)|
|%i| Минуты (от 00 до 59)|
|%s| Секунды (от 00 до 59)|
|%f| Микросекунды (от 000000 до 999999)|
|%p| A.M. или P.M.|

Для текущего времени можно использовать:
```SQL
SELECT CURRENT_DATE(), CURRENT_TIME(), CURRENT_TIMESTAMP();
```

## Работа с времеными данными

### Временные функции возвращающие даты

Добавление интервала времени:
```SQL
SELECT DATE_ADD(CURRENT_DATE(), INTERVAL 5 DAY);
```

Типы интервалов:
|Интервал| Описание|
|-|-|
|Second| Количество секунд|
|Minute| Количество минут|
|Hour| Количество часов|
|Day| Количество дней|
|Month| Количество месяцев|
|Year| Количество лет|
|Minute_second| Количества минут и секунд, разделенные двоеточием|
|Hour_second| Количества часов, минут и секунд, разделенные двоеточием|
|Year_month| Количества лет и месяцев, разделенные дефмсом|

Функция `LAST_DAY()` — определяет последний день месяца:
```SQL
SELECT LAST_DAY('2005-03-25');
```

Функция `CONVERT_TZ()` преобразует время одного часового пояса в другой:
```SQL
SELECT CONVERT_TZ(CURRENT_TIMESTAMP(), 'US/Eastern' 'UTC');
```

### Функции возвращающие строки

`DAYNAME()` — возвращает имя дня недели:
```SQL
SELECT DAYNAME('2005-03-22');
```

`EXTRACT()` извлекает из времени и даты составные части, использует теже интервалы, что и `DATE_ADD()`:
```SQL
SELECT EXTRACT(YEAR FROM '20050322 22:19:05');
```

### Функции возвращающие числа

`DATEDIFF()` — возвращает количество полных дней между датами:

```SQL
SELECT DATEDIFF('2005-09-05', '2005-06-22');
```
Если первая дата раньше, то вернётся отрицательное число. Функция не учитывает время.

## Функции преобразования

Для преобразования используется `CAST()`:
```SQL
SELECT CAST('1456328' AS SIGNED INTEGER);
```
При преобразовании строки функция будет преобразовывать числа строке слева направа до первого нечислового символа.

При преобзазовании строк в даты через функции `CAST()` нужно использовать стандартные форматы временных данных, для иных случаев нужно использовать функцию `STR_TO_DATE()`.
