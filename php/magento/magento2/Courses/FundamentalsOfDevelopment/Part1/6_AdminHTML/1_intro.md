# Admin HTML

## Назначение

* Позволяет клиентам управлять сущностями мадженты, такими как продукты, категории, ценовые правила, заказы и т.д. и т.п.
* Предоставляет интерфейс для конфигурирования


## Контроллеры

Все маджентовские контролеры админки наследуются от [`Magento\Backend\App\Action`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Backend/App/Action.php) который наследуется от [`Magento\Backend\App\AbstractAction`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Backend/App/AbstractAction.php).

## Гриды и формы

Гриды и формы реализованы через UiComponents.

## Системные конфигурации

Все значения системных конфигураций хранятся в БД, с разными скопами: global, website, store. 

Модифицировать конфигурацию можно посрдеством XML-конфигов.