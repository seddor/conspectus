# Aliases

[Aliases](https://getcomposer.org/doc/articles/aliases.md)

В composere для веток можно делать алиасы, которые можно использовать потом в качечтва номера версии, например
```json
{
    "extra": {
        "branch-alias": {
            "dev-master": "1.0.x-dev"
        }
    }
}
```
тогда при запроса 1.0.* будет отдваться последняя версия ветки `master`.

## инлайновые алиасы

так-же можно указать чтобы все зависимости ставились из определённой ветки, вместо той, которая у них указана:
```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/you/monolog"
        }
    ],
    "require": {
        "symfony/monolog-bundle": "2.0",
        "monolog/monolog": "dev-bugfix as 1.0.x-dev"
    }
}
```
или
```bash
php composer.phar require monolog/monolog:"dev-bugfix as 1.0.x-dev"
```
здесь `symfony/monolog-bundle` требует `monolog/monolog` версии `1.*`, благодаря алиасу вместо неё будет поставлена из версия из ветки `bugfix`.
