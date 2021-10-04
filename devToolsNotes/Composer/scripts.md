# Scripts

[Scripts](https://getcomposer.org/doc/articles/scripts.md)

composer позволяет запускать какие-либо PHP скрпиты в процеса своей работы, запуск скриптов можно быть привязан к определённому событию.

## События команд
* `pre-install-cmd`
* `post-install-cmd`
* `pre-update-cmd`
* `post-update-cmd`
* `post-status-cmd`
* `pre-archive-cmd`
* `post-archive-cmd`
* `pre-autoload-dump`
* `post-autoload-dump`
* `post-root-package-install`
* `post-create-project-cmd`
## События установки
* `pre-dependencies-solving`
* `post-dependencies-solving`
## События пакетов
* `pre-package-install`
* `post-package-install`
* `pre-package-update`
* `post-package-update`
* `pre-package-uninstall`
* `post-package-uninstall`
## События плагинов
* `init`
* `command`
* `pre-file-download`
* `pre-command-run`

## Определение скриптов

```json
{
    "scripts": {
        "post-update-cmd": "MyVendor\\MyClass::postUpdate",
        "post-package-install": [
            "MyVendor\\MyClass::postPackageInstall"
        ],
        "post-install-cmd": [
            "MyVendor\\MyClass::warmCache",
            "phpunit -c app/"
        ],
        "post-autoload-dump": [
            "MyVendor\\MyClass::postAutoloadDump"
        ],
        "post-create-project-cmd": [
            "php -r \"copy('config/local-example.php', 'config/local.php');\""
        ]
    }
}
```
Скрипт выглядит примерно так:
```php
<?php

namespace MyVendor;

use Composer\Script\Event;
use Composer\Installer\PackageEvent;

class MyClass
{
    public static function postUpdate(Event $event)
    {
        $composer = $event->getComposer();
        // do stuff
    }

    public static function postAutoloadDump(Event $event)
    {
        $vendorDir = $event->getComposer()->getConfig()->get('vendor-dir');
        require $vendorDir . '/autoload.php';

        some_function_from_an_autoloaded_file();
    }

    public static function postPackageInstall(PackageEvent $event)
    {
        $installedPackage = $event->getOperation()->getPackage();
        // do stuff
    }

    public static function warmCache(Event $event)
    {
        // make cache toasty
    }
}
```

## Классы событий

* Base class: [Composer\EventDispatcher\Event](https://getcomposer.org/apidoc/master/Composer/EventDispatcher/Event.html)
* Command Events: [Composer\Script\Event](https://getcomposer.org/apidoc/master/Composer/Script/Event.html)
* Installer Events: [Composer\Installer\InstallerEvent](https://getcomposer.org/apidoc/master/Composer/Installer/InstallerEvent.html)
* Package Events: [Composer\Installer\PackageEvent](https://getcomposer.org/apidoc/master/Composer/Installer/PackageEvent.html)
* Plugin Events:
  * init: [Composer\EventDispatcher\Event](https://getcomposer.org/apidoc/master/Composer/EventDispatcher/Event.html)
  * command: [Composer\Plugin\CommandEvent](https://getcomposer.org/apidoc/master/Composer/Plugin/CommandEvent.html)
  * pre-file-download: [Composer\Plugin\PreFileDownloadEvent](https://getcomposer.org/apidoc/master/Composer/Plugin/PreFileDownloadEvent.html)

## Запуск вручную 

```bash
composer run-script [--dev] [--no-dev] script
```

## Кастмные скрипты

Так-же можно писать кастомные скрипты, [подробнее](https://getcomposer.org/doc/articles/scripts.md#writing-custom-commands)
