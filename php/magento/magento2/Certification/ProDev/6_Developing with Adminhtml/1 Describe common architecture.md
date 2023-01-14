# Describe common structure/architecture

>Describe the difference between Adminhtml and frontend. What additional tools and requirements exist in the admin?

Как и в M1, в M2 имеется панель администрирования. Основной модуль админки [`Magento_Backend`](https://github.com/magento/magento2/tree/2.4/app/code/Magento/Backend).

Для конфигурации админки действует специфичная область (area): `adminhtml`, также админка считается отдельным стором с ИД = 0.

Роутер админки: [`Magento\Backend\App\Router`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/App/Router.php), он наследует [`Magento\Framework\App\Router\Base`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Router/Base.php), сама логика роутера берётся из базового, в роуте админки добавляется префикс и изменяются некоторые параметры роутера.

Экшен контроллеры админки должны наследоваться от контролера [`Magento\Backend\App\Action`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/App/Action.php) он наследует [`Magento\Backend\App\AbstractAction`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/App/AbstractAction.php). Сам класс `Action` не содержит ничего, вся логика находится в `AbstractAction`.

## Describe the difference between Adminhtml and frontend

* ACL — в админке есть механизм ределения доступа к ресурсам админки
* Другие базоваые классы:
  * Контроллер: [`Magento\Backend\App\Action`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/App/Action.php)
  * Блок: [`Magento\Backend\Block\AbstractBlock`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/Block/AbstractBlock.php)
  * Модель для URL: [`Magento\Backend\Model\Url`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/Model/Url.php)
* Пути в админке содержат префикс `admin` (или другой из настройки).
