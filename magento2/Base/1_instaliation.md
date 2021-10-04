# Magento 2 installtion

[Install Magento using Composer (2.3)](https://devdocs.magento.com/guides/v2.3/install-gde/composer.html)

[Install Magento using Composer (2.4)](https://devdocs.magento.com/guides/v2.4/install-gde/composer.html)

## Понадобится:

* Аккаунт [Magento marketplace](https://marketplace.magento.com/)
* [Сomposer](https://getcomposer.org/)
* **(только для 2.4)** [Elasticsearch](https://www.elastic.co/)

## Установка

1. Запустить в корне проекта:
```bash
composer create-project --repository=https://repo.magento.com/ magento/project-community-edition .
```
2. Ввести ключи доступа [отсюда](https://marketplace.magento.com/customer/accessKeys/)

### Magento 2.4

### Elasticsearch

[Elasticsearch (документация Magento по установке)](https://devdocs.magento.com/guides/v2.4/install-gde/prereq/elasticsearch.html)

Начиная с 2.4 Magento перестаёт использовать поиск через MySQL, вместо него используется Elasticsearch

### Установка через консоль

[Install the Magento software using the command line](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli.html)

С версии 2.4 больше не доступна установка через браузер, вместо неё нужно спользовать установку через консоль, команда:

```bash
php bin/magento setup:install -vvv --db-name=magento24 --db-user=<DB_USER> --db-password=<DB_PASS> --admin-user=admin --admin-password=admin123 --admin-email=<ADMIN_EMAIL> --admin-firstname=admin --admin-lastname=admin
```

### Elasticsearch

Начиная с 2.4 обязательно установить Elasticsearch

## Nginx

* `conf.d/upstreams.conf`

```
upstream fastcgi_backend {
    #127.0.0.1:9072
    server unix:/var/run/php/php7.2-fpm.sock;
}
```

* `<virtual_host>.conf`

server {
    listen          80;
    server_name     magento2.local;
    
    set $MAGE_ROOT /var/www/magento2;

    access_log /var/log/nginx/magento2.access.log;
    error_log /var/log/nginx/magento2.error.log;

    include         /var/www/magento2/nginx.conf.sample;
}

## Режимы

[Set the Magento mode](https://devdocs.magento.com/guides/v2.4/config-guide/cli/config-cli-subcommands-mode.html)

В m2 добавили режимы, по умолчанию маджента находиться в `default` моде.

Для разработки стоит переключить в мод `developer`

```bash
bin/magento deploy:mode:set developer
```

## Sample data

[Installing sample data using Composer](https://devdocs.magento.com/guides/v2.4/install-gde/install/sample-data-after-composer.html)

```bash
bin/magento sampledata:deploy
bin/magento setup:upgrade
```
Может возникнуть проблема нападобии:
```
PHP message: PHP Fatal error:  Uncaught ReflectionException: Class Magento\Framework\App\ResourceConnection\Proxy does not exist
```
Стоит очистить директорию `generated/`
