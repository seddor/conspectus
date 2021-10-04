# Utilize the CLI

>Describe the usage of `bin/magento` commands in the development cycle. Which commands are available? How are commands used in the development cycle?

>Demonstrate an ability to create a deployment process. How does the application behave in different deployment modes, and how do these behaviors impact the deployment approach for PHP code, frontend assets, etc.?

## Describe the usage of `bin/magento` commands in the development cycle. 

### Which commands are available? How are commands used in the development cycle?

* `setup:di:compile` — генерирует код (прокси, фабрики и т.п.) и метаданные для DI, при использовании стоит учитывать, что при наличии метаданных DI не будет генерироватся автоматически
* `setup:static-content:deploy` — деплой статичных файлов в _pub/static_
*  `dev:source-theme:deploy` — генерирует CSS из LESS
* `cache:clean` и `cache:flush` — очищает кеш мадженты и хранилища соответственно
* `maintenance:enable` и `maintenance:disable` — включает/отключает [maintenance mode](https://devdocs.magento.com/guides/v2.4/install-gde/install/cli/install-cli-subcommands-maint.html)

## Добавление CLI-команды

[How to add CLI commands](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/cli-cmds/cli-howto.html)
[Репозиторй от маджеты с примером команды](https://github.com/magento/magento2-samples/tree/master/sample-module-command)

В M2 CLI-команды базируются на [Symfony Console component](https://symfony.com/doc/current/components/console.html)

Для создания новой команды нужно:
1. Создать класс команды в _`<module-dir>`/Console/Command_, класс должен наследоваться от [`Symfony\Component\Console\Command\Command`](https://github.com/symfony/console/blob/master/Command/Command.php)
```php
<?php
    namespace Magento\CommandExample\Console\Command;

    use Symfony\Component\Console\Command\Command;
    use Symfony\Component\Console\Input\InputInterface;
    use Symfony\Component\Console\Input\InputOption;
    use Symfony\Component\Console\Output\OutputInterface;

    /**
     * Class SomeCommand
     */
    class SomeCommand extends Command
    {
        /**
         * @inheritDoc
         */
        protected function configure()
        {
            $this->setName('my:first:command');
            $this->setDescription('This is my first console command.');

            parent::configure();
        }

        /**
         * @param InputInterface $input
         * @param OutputInterface $output
         *
         * @return null|int
         */
        protected function execute(InputInterface $input, OutputInterface $output)
        {
            $output->writeln('<info>Success Message.</info>');
            $output->writeln('<error>An error encountered.</error>');
        }
    }
```
2. Добавить класс в DI, в `di.xml`:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <type name="Magento\Framework\Console\CommandList">
        <arguments>
            <argument name="commands" xsi:type="array">
                <item name="commandexample_somecommand" xsi:type="object">Magento\CommandExample\Console\Command\SomeCommand</item>
            </argument>
        </arguments>
    </type>
</config>
```
3. Почистить кеш:
```bash
cd <magento_root>/var
rm -rf cache/* page_cache/* di/* generation/*
```

## Опции и аргуметны

* Опции имеют флаг — `bin/magento do:something --admin-email="myemail@swiftotter.com"`
* аргументы не имеют флага — `bin/magento do:something myemail@swiftotter.com`

## Окружение

Задать окружение можно с помощью [`Magento\Framework\App\State::setAreaCode()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/State.php#L131).

Если окружение нужно изменить только для части кода то нужно пользоватся эмуляцией.

### Эмуляция

#### Эмуляция области (area)

Для эмуляции области можно спользовать [`Magento\Framework\App\State::emulateAreaCode()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/State.php#L179), метод применимает код нужной области, выполняемые функцию и массив её аргументов.

Если нужно вытащить конфигурацию из другой области, то такая эмуляция не подойдёт, т.к. конфигурация грузиятся по текущему scope из [`Magento\Framework\Config\ScopeInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Config/ScopeInterface.php), который эмуляцией не перезаписвается: [`Magento\Framework\Config\Data\Scoped::_loadScopedData()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Config/Data/Scoped.php#L106):
```php
protected function _loadScopedData()
{
    $scope = $this->_configScope->getCurrentScope();
```

В таком случае можно сделать так ([`Magento\Config\Console\Command\EmulatedAdminhtmlAreaProcessor::process`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Config/Console/Command/EmulatedAdminhtmlAreaProcessor.php#L53)):
```php
$currentScope = $this->scope->getCurrentScope();
try {
    return $this->state->emulateAreaCode(Area::AREA_ADMINHTML, function () use ($callback, $params) {
        $this->scope->setCurrentScope(Area::AREA_ADMINHTML);
        return call_user_func_array($callback, $params);
    });
} catch (\Exception $exception) {
    throw $exception;
} finally {
    $this->scope->setCurrentScope($currentScope);
}
```

#### Эмкляция стора

Если нужно эмулировать конкретный стор, то можно использовать [`Magento\Store\Model\App\Emulation`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Store/Model/App/Emulation.php), запускается примерно так:
```php
try {
    $this->appEmulation->startEnvironmentEmulation($storeId);
    //code
} finally {
    $this->appEmulation->stopEnvironmentEmulation();
}
```

[`startEnvironmentEmulation()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Store/Model/App/Emulation.php#L116):
* в storeManager ставится переданный `$storeId`
* ставится дизайн по `$area` (передаётся вторым аргуметом, по умолчанию frontend) и `$storeId`
* настраиваются переводы по `$storeId`

#### Эмуляция всего вместе

Чтобы эмулировать область и стор одновременно нужно использовть оба метода одновременно.
