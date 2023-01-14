# Метадата

По сути метадата представляет собой данные об объектах БД. 

Разные СУБД хранят и работают с метадатой по разному. В MySQL хранит её в специальной БД `information_schema`.

## information_schema

MySQL хранит информацию об объектах базы данных в специальной БД `information_schema`, информацию из этой базы можно запрашивать как из других баз:
```SQL
SELECT table_name, table_type
    FROM information_schema.tables
    WHERE table_schema = 'bank'
```
* tables — хранит информацию о таблицах
* views — хранит информацию о представлениях
* columns — о 
* statistics — информация об индексах
* schemata — базы данных
* 
|Таблица|Информация о|
|-|-|
|schemata|базах данных|
|tables|таблицах|
|views|представлениях|
|columns|столбцах таблиц и представлений|
|statistics|индексах|
|User_Privileges|привелегии на объекты схемы|
|Schema_Privileges|привелении на базы данных|
|Table_Privileges|привелнии на таблицы|
|Column_Privileges|привелегии на столбцы|
|Character_Sets|доступные чарсеты|
|Collations|Методы сравнения доступные для чарсетов|
|Collation_Character_Set_Applicability|Чарсеты доступные для каждого метода сравнения|
|Table_Constraints|Уникальные, внешние, первичные ограничения|
|Key_Column_Usage|отношения ограничений и ключей|
|Routines|функции и процедуры|
|Triggers|тригеры таблиц|
|Plugins|плагина СУБД|
|Engines|доступные механизмы хранения|
|Partitions||
|Events||
|Process_List|Запущеные процессы|
|Referential_Constraints|внешние ключи|
|Global_Status|Информация о статусе сервера|
|Session_Status|Информация о статусе сессии|
|Global_Variables|глобальные переменные|
|Session_Variables|сессионные переменные|
|Parameters|хранимые параметры процедур и функций|
|Profiling||

