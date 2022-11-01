# Кеш

Кеширование реализовано в копоненте [`Magento\Freamwork\Cache`](https://github.com/magento/magento2/tree/2.3/lib/internal/Magento/Framework/Cache) который наследуется от `Zend_Cache`.

## Конфигурация

Дефолтный список типов кешей находится в _app/etc/env.php_.

## Типы

Типы кеша группируют данные кеша на основе функциональной роли. Операци очистки, включения/выключения кеша, могут быть ограничены определённым типом. Управлять кешом можно через амдинку (в production режиме через админку можно только чистить кеш) или через cli `bin/magento`.

## Очистка

Именются 3 способа очистки кеша:

* Через админку
* Используя команду: `php bin/magento cache:clean`
* Ручное удления кеша, например для кеша в файлах командой: `rm -rf var/cache/*`