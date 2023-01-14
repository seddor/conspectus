# The composer.json Schema

[The composer.json Schema](https://getcomposer.org/doc/04-schema.md)
[описание схемы в json формате](https://getcomposer.org/schema.json)

## Свойства

### name

Имя пакета предстваяет из себя `<vendor_name>/<project_name>`.

### description

Краткое описание пакета.

### version

версия пакета, в формете `X.Y.Z` или `vX.Y.Z`, может включать суфиксы `-dev`, `-patch` (`-p`), `-alpha` (`-a`), `-beta` (`-b`) or `-RC`.

версия опциональна, если в пакете используется система контроля версий, в таком случае версия может быть взята из тегов репозитория.

### type

Тип пакета. По умолчанию `library`.

Используется для кастомизации логики инсталяции пакета. При необходимости можно определять кастомные типы, например в Magento 2 для модулей используется тип `m2-module`.

Типы доступные по умолчанию:
* `libary` — используется по умолчанию, просто устанавливает зависимость в `vendor`.
* `project` — проекты отличные от билиотек, например [Symfony Standard Edition](https://github.com/symfony/symfony-standard) или [SilverStripe installer](https://github.com/silverstripe/silverstripe-installer)
* `metapackage` — пустой пакет, содержащий список зависимостец и обеспечивающий их установку.
* `composer-plugin` — пакеты, которые могут предоставлять инсталер для других пакетов с кастомными типами.

[Setting up and using custom installers](https://getcomposer.org/doc/articles/custom-installers.md)

### keywords

ключевые слова для пакета, используются для поиска.

Опциональный.

### homepage

URL сайта проекта.

Опциональный.

### readme

Относительный путь к readme.

Опциональный.

### time

Время релиза версии, может быть в формате `YYYY-MM-DD` или `YYYY-MM-DD HH:MM:SS`.

Опциональный.

### license

Лецензия для пакета.

[Список лицензий](https://spdx.org/licenses/)

Опциональный, но рекомендуемый.

### authors

Авторы пакета, представляет из себя массив объектов.

Объект содержит такие свойства (опциональны):

* `name`
* `email`
* `homepage`
* `role` — роль в проекте (например developer или translator).

Опциональный, но рекомендуемый.

### support

Информация по поддержке.

Может включать:

* `email`
* `issues`
* `forum`
* `wiki`
* `irc`
* `source`
* `docs`
* `rss`
* `chat`

Опциональный.

## Пакеты

### require

Список зависимостей.

### require-dev

Список dev-зависимотей.

### conflict

Список пакетов, которые конфликтуют с этим пакетом.

### replace

Список пакетов заменяемых данным пакетом, можно использовать для форков.

### suggest

Список опциональных зависимосетей пакетов.

## autoload

Автолоадинг, может использовать [`psr-4`](https://www.php-fig.org/psr/psr-4/), [`psr-0`](https://www.php-fig.org/psr/psr-0/), `classmap`, `files`.

Так-же можно определить `autoload-dev`.

## minimum-stability

Определяет дефолтное поведение для фильтрации пакетов по стабильности.

Доступные значения:

* `dev`
* `alpha`
* `beta`
* `RC`
* `stable`

## prefer-stable

Предпочитать стабильные версии пакетов.

## repositories 

Кастомные резопитории, представляет из себя массив объектов.

Поддерживаемые типы:
* `composer`
* `vcs`
* `pear`
* `package`

Пример:
```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "http://packages.example.com"
        },
        {
            "type": "composer",
            "url": "https://packages.example.com",
            "options": {
                "ssl": {
                    "verify_peer": "true"
                }
            }
        },
        {
            "type": "vcs",
            "url": "https://github.com/Seldaek/monolog"
        },
        {
            "type": "pear",
            "url": "https://pear2.php.net"
        },
        {
            "type": "package",
            "package": {
                "name": "smarty/smarty",
                "version": "3.1.7",
                "dist": {
                    "url": "https://www.smarty.net/files/Smarty-3.1.7.zip",
                    "type": "zip"
                },
                "source": {
                    "url": "https://smarty-php.googlecode.com/svn/",
                    "type": "svn",
                    "reference": "tags/Smarty_3_1_7/distribution/"
                }
            }
        }
    ]
}
```

## config

[Конфигурации](https://getcomposer.org/doc/06-config.md)

## scripts

[Скрипты](https://getcomposer.org/doc/articles/scripts.md) запускаемые при различных событиях. 

## extra

Дополнительные данные доступные в `scripts`.

В скриптах можно использовать так:
```php
$extra = $event->getComposer()->getPackage()->getExtra();
```

## bin

Список файлов, которые используются как бинарники и на них создаются симлинки в `bin-dir` (из конфига).

[Vendor binaries and the `vendor/bin` directory](https://getcomposer.org/doc/articles/vendor-binaries.md)

## archive

Опции используемые при создании архива для пакета. Здесь можно указать список исключений:
```json
{
    "archive": {
        "exclude": ["/foo/bar", "baz", "/*.test", "!/foo/bar/baz"]
    }
}
```

## abandoned

Индикатор того, что пакет заброшен.

Значения:
* `"abandoned": true` — пакет заброшен
* `"abandoned": "monolog/monolog"` — пакет заброшен, в качестве альтернативые рекомендуется использовать `monolog/monolog`.

Опциональный.

