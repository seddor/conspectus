# Utilize ACL to set menu items and permissions

>Describe how to set up a menu item and permissions. How would you add a new menu item in a given tab? How would you add a new tab in the Admin menu? How do menu items relate to ACL permissions?

>Describe how to check for permissions in the permissions management tree structures. How would you add a new user with a given set of permissions? How can you do that programmatically?

# Describe how to set up a menu item and permissions. How would you add a new menu item in a given tab? How would you add a new tab in the Admin menu? How do menu items relate to ACL permissions?

>Describe how to set up a menu item and permissions.

Настроить ACL для элмента меню можно в `menu.xml`
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Backend:etc/menu.xsd">
    <menu>
        <!-- таба -->
        <add
                id="Oggetto_SkinCare::skin_care"
                title="Skin consult"
                translate="title"
                module="Oggetto_SkinCareDiagnostic"
                parent="Magento_Backend::marketing"
                sortOrder="100"
                resource="Oggetto_SkinCare::skin_care"
        />
        <!-- элемент меню, будет находится в табе -->
        <add
                id="Oggetto_SkinCareMgmLink::skin_care_mgm_link"
                title="MGM links"
                translate="title"
                sortOrder="20"
                module="Oggetto_SkinCareMgmLink"
                parent="Oggetto_SkinCare::skin_care"
                action="skin_care_mgm_link/link/index"
                dependsOnModule="Oggetto_SkinCare"
                resource="Oggetto_SkinCareMgmLink::skin_care_mgm_link"
        />
    </menu>
</config>
```
ACL указывается в `resource`.

>How would you add a new menu item in a given tab

Нужно у меню указать соответствующий `parent`.

>How would you add a new tab in the Admin menu?

Нужно добавить новый айтем меню, у которого не указывать `parent` и `action`.

>How do menu items relate to ACL permissions?

ACL для айтема зависит от ресруса, который указан у него в `resource`. Проверка происходит в [`Magento\Backend\Model\Menu\Item::isAllowed()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Backend/Model/Menu/Item.php#L451)

# Describe how to check for permissions in the permissions management tree structures. How would you add a new user with a given set of permissions? How can you do that programmatically?

>How would you add a new user with a given set of permissions?

* _System — Permissions — User roles_ — содержит все роли, для роли указываются доступные для неё пермишены.
* _System — Permissions — All users_ — содержит всех пользователей админки, у пользователя указывается роль.


>How can you do that programmatically?

Модель для пользователя: [`Magento\User\Model\User`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/User/Model/User.php) можно создать её инстанс, проставить нужные данные и сохранить.
```php
$adminUser->setRoleId($role->getId())
    ->setEmail('admin' . $i . '@example.com')
    ->setFirstName('Firstname')
    ->setLastName('Lastname')
    ->setUserName('admin' . $i)
    ->setPassword('123123q')
    ->setIsActive(1);
```
Для роли используется класс [`Magento\Authorization\Model\Role`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Authorization/Model/Role.php)


## ACL

ACL настраивается в _`<module-dir>`/etc/acl.xml_:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Acl/etc/acl.xsd">
    <acl>
        <resources>
            <resource id="Magento_Backend::admin">
                <resource id="Ewave_Utilities::ewave">
                    <resource id="Ewave_ProductCalculator::calculator">
                        <resource id="Oggetto_ProductCalculatorDiagnostic::symptom" sortOrder="40" title="Manage symptoms" />
                        <resource id="Oggetto_ProductCalculatorDiagnostic::symptom_value" sortOrder="41" title="Manage symptoms values" />
                    </resource>
                </resource>
            </resource>
        </resources>
    </acl>
</config>
```

Для проверки доступа используется метод [`Magento\Framework\Authorization::isAllowed()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Authorization.php#L45)

### WebAPI

ACL также используется в API, для каждого роута API можно указать ресурс:
```xml
<route url="/V1/acl" method="GET">
    <service class="Chapter5\ACL\Api\AclInterface" method="getById"/>
    <resources>
        <resource ref="Chapter5_ACL::all_actions"/>
    </resources>
</route>
```
Для проверки доступа используется метод [`Magento\Framework\Webapi\Authorization::isAllowed()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Webapi/Authorization.php#L39). 

API так-же подерживает такие виды авторизации:
* `anonymous`
* `self` — использует фронтенд сессию пользователя.

Для доступа к админкским ресурсам нужно получать токен.

Для аутентификации можно использовать **Oauth 1.0**

[Authentication](https://devdocs.magento.com/guides/v2.4/get-started/authentication/gs-authentication.html)
[Get the admin token](https://devdocs.magento.com/guides/v2.4/rest/tutorials/orders/order-admin-token.html)
