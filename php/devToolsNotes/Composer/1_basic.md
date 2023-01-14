# [Basic usage](https://getcomposer.org/doc/01-basic-usage.md)

## `composer.json`

`composer.json` — файл содержащий описание зависимостей и прочие методанные необходимые для работы проекта.

### `require`

Секция `require` содержит описание завимостей:

```json
{
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
```

`monolog/monolog` — имя пакета, `1.0.*` требуемая версия. 

Имя пакета состоит из `<имя_вендора>/<имя_проекта>`.

[Подробнее о номерах версий](https://getcomposer.org/doc/articles/versions.md)

Пакеты ставятся или из основного репозитория [Packagist](https://packagist.org/) или из репозиториев, которые указываются в разделе `repositories`.

## Установка зависимостей

Для установки зависимостей используется команда:

```bash
composer install
```

после её запукса, будут скачены зависимости и помещены в директорию `vendor` в корне проекта, будет сгенирован(если не было) `composer.lock`. 

### `composer.lock`

Содержит указания конкретные версии пакетов, с которыми была произведена установка, при его наличии `coposer install` будет устанавливать версии указаные в `coposer.lock`, если нужно обновить все версии до максимально доступных из описания в `composer.json` используется `composer update`.

### Пакеты платформы

В composer есть виртуальные пакеты, которые не устанавливаются через сам composer, но могут быть указаны в качестве зависимости и их наличие будет проверятся в системе при попытке запуска `install/update`:

* `php`
* `hhvm`
* `ext-<name>`
* `lib-<name>`
С помощью команды `composer show --platform` можно увидеть пакеты доступные на локалке.

## Автолоадинг

`composer` предоставляет автолоадинг установленых пакетов, для этого он генерирует `vendor/autoload.php`. Который можно включить в нужное место для доступа к пакетам:

```php
require_once __DIR__ . '/vendor/autoload.php';
```

Управлять автолоадом можно в разделе `autoload`:

```json
{
    "autoload": {
        "psr-4": {"Acme\\": "src/"}
    }
}
```

таким обрзаом composer зарегистрирует [PSR-4](https://www.php-fig.org/psr/psr-4/) автолоадинг для неймспейса `Acme`.

После измения `autoload` нужно запустить `composer dump-autoload` для перегенерации `vendor/autoload.php`.