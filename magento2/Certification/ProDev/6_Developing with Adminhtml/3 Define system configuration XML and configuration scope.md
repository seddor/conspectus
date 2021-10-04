# Define system configuration XML and configuration scope

>Define basic terms and elements of system configuration XML, including scopes. How would you add a new system configuration option? What is the difference in this process for different option types (secret, file)?

>Describe system configuration data retrieval. How do you access system configuration options programmatically?

# Define basic terms and elements of system configuration XML, including scopes. How would you add a new system configuration option? What is the difference in this process for different option types (secret, file)?

Системные конфигурации хранятся в базе в таблице `core_config_data`. Каждая конфигурация может иметь свой скоуп применения.

Системные конфиги настраиваются в `system.xml` файлах. Они находятся по пути _`<module_dir>`/etc/adminhtml/system.xml_ (подробнее по system.xml в курсах, 1.6.2)

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <tab id="skin_care" translate="label" sortOrder="1000">
            <label>Skin Care</label>
        </tab>
        <section id="skin_care_mgm" translate="label" type="text" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="1">
            <class>separator-top</class>
            <label>MGM</label>
            <tab>skin_care</tab>
            <resource>Oggetto_SkinCareMgmLink::config</resource>
            <group id="link" translate="label" type="text" sortOrder="20" showInDefault="1" showInWebsite="1" showInStore="1">
                <label>Link</label>
                <field id="enabled" translate="label comment" type="select" sortOrder="0" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="0">
                    <label>Enabled</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="prefix" translate="label comment" type="text" sortOrder="1" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="0">
                    <label>Prefix</label>
                    <validate>required-entry</validate>
                </field>
                <field id="redirect_path" translate="label comment" type="text" sortOrder="2" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="0">
                    <label>Redirect to path</label>
                    <validate>required-entry</validate>
                </field>
            </group>
        </section>
    </system>
</config>
```

Путь значения конфига обычно состоит из 3х элементов элементов, например `skin_care_mgm/link/enabled`:

* skin_care_mgm — раздел (элемент `section`), он отображется в админке в панеле всех настроек, для раздела настраивается ACL ресурс, указывается tab.
* link — группа (элемент `group`), отображается на странице секции. 
* enabled — поле (элемент `field`), отображается на странице секции, в группе.

В конфигурации поддерживаются вложенные группы, поэтому количество элементов `group` будет равно вложености поля.

# Describe system configuration data retrieval. How do you access system configuration options programmatically?

Для извлечения данных конфигуации используется класс [`Magento\Framework\App\Config`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Config.php) который реализует [`Magento\Framework\App\Config\ScopeConfigInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Config/ScopeConfigInterface.php), он предоставляет два метода:
```php
/**
 * Retrieve config value by path and scope.
 *
 * @param string $path The path through the tree of configuration values, e.g., 'general/store_information/name'
 * @param string $scopeType The scope to use to determine config value, e.g., 'store' or 'default'
 * @param null|int|string $scopeCode
 * @return mixed
 */
public function getValue($path, $scopeType = ScopeConfigInterface::SCOPE_TYPE_DEFAULT, $scopeCode = null);

/**
 * Retrieve config flag by path and scope
 *
 * @param string $path The path through the tree of configuration values, e.g., 'general/store_information/name'
 * @param string $scopeType The scope to use to determine config value, e.g., 'store' or 'default'
 * @param null|int|string $scopeCode
 * @return bool
 */
public function isSetFlag($path, $scopeType = ScopeConfigInterface::SCOPE_TYPE_DEFAULT, $scopeCode = null);
```

[`getValue()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Config.php#L56) извлекает данные, [`isSetFlag()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Config.php#L91) извлекает данные через `getValue()` и приводит их к bool.

Для извлечения данных используется класс [`Magento\Config\App\Config\Type\System`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Config/App/Config/Type/System.php) реализующий [`Magento\Framework\App\Config\ConfigTypeInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/App/Config/ConfigTypeInterface.php)
