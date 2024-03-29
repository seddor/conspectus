# URL реврайты

M2 позволяет задавать URL key (URL идентификатор) для каждой статичной, контентной, продуктовой и категорийной страницы.

## Флоу

1. Запрос обрабатывается в роутере [`urlrewrite`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/UrlRewrite/Controller/Router.php)
2. Роутер ищет реврайты в таблице `url_rewrite`

### Таблица `url_rewrite`

Структура:
```
+------------------+----------------------+------+-----+---------+----------------+
| Field            | Type                 | Null | Key | Default | Extra          |
+------------------+----------------------+------+-----+---------+----------------+
| url_rewrite_id   | int(10) unsigned     | NO   | PRI | NULL    | auto_increment |
| entity_type      | varchar(32)          | NO   |     | NULL    |                |
| entity_id        | int(10) unsigned     | NO   |     | NULL    |                |
| request_path     | varchar(255)         | YES  | MUL | NULL    |                |
| target_path      | varchar(255)         | YES  | MUL | NULL    |                |
| redirect_type    | smallint(5) unsigned | NO   |     | 0       |                |
| store_id         | smallint(5) unsigned | NO   | MUL | NULL    |                |
| description      | varchar(255)         | YES  |     | NULL    |                |
| is_autogenerated | smallint(5) unsigned | NO   |     | 0       |                |
| metadata         | varchar(255)         | YES  |     | NULL    |                |
+------------------+----------------------+------+-----+---------+----------------+
```
