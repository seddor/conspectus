# Типы MySQL

## Charset

Может хранить строку фиксированной длины:

* `char(<length>)` — строка длиной до 255байт.
* `varchar(<length>)` — строка до 65535 байт(с 5й версии).

## Text

* `tinytext` — 255байт
* `text` — 65,535байт
* `mediumtext` — 16,777,215байт
* `longtext` — 4,294,967,295байт

Особености:

* Если размер данных загружаемых в столбец будет больше, то данные обрежутся.
* Не удаляет конечные пробелы(`varchar` удаляет).
* При использовании сортировок для текстоквых полей сортировки идёт по первым 1024байтам, при необходимости можно увеличить.

## Numeric data

Для чисел можно укзать модификатор `UNSIGNED` и тогда число будет использоваться без знака.

Целые числа:

|Тип|Длина(со знаком)|Длина(без знака)|
|tinyint|−128 to 127|0 to 255|
|smallint|−32,768 to 32,767|0 to 65,535|
|mediumint|−8,388,608 to 8,388,607|0 to 16,777,215|
|int|−2,147,483,648 to 2,147,483,647|0 to 4,294,967,295|
|bigint|−9,223,372,036,854,775,808 to 9,223,372,036,854,775,807|0 to 18,446,744,073,709,551,615|

Дробные числа:

|Тип|Длина|
|Float(p,s)|−3.402823466E+38 to −1.175494351E-38
and 1.175494351E-38 to 3.402823466E+38|
|Double(p,s)|−1.7976931348623157E+308 to −2.2250738585072014E-308
and 2.2250738585072014E-308 to 1.7976931348623157E+308|

* `p (percision)` — точность, количество цифр слева
* `s (scale)` — масштаб, количество цифр справа

Числа с большим количенством цифр, после запятой, будут округлятся автоматически при сохранении. 

## Temporal data

|Тип|Дефолтный формат|Допутимые значения|
|date|YYYY-MM-DD|1000-01-01 to 9999-12-31|
|datetime|YYYY-MM-DD HH:MI:SS|1000-01-01 00:00:00 to 9999-12-31 23:59:59|
|timestamp|YYYY-MM-DD HH:MI:SS|1970-01-01 00:00:00 to 2037-12-31 23:59:59|
|year|YYYY|1901 to 2155|
|time|HH:MI:SS|−838:59:59 to 838:59:59|