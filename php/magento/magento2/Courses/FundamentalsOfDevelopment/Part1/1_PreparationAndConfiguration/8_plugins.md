# Плагины

Плагины используются для расширения (изменения) поведение паблик методов класса. Плагины изменяют *поведение* классов, но не изменяют сам класс.

Плагины нельзя использовать для финальных методов и классов, пиватных методов или классов созданых бех использования DI. Для того чтобы плагины работали корректно нужно правильно именовать классы и методы.

Плагины можно вызывать до, после или вместо целевого метода, при этом плагины не конфликтуют т.к. вызываются в определённом порядке.

## Определение плагина

Плагины определяются в `di.xml`:

```xml
<type name="Magento\Theme\Block\Html\Topmenu">
    <plugin name="catalogTopmenu" type="Magento\Catalog\Plugin\Block\Topmenu" sortOrder="1" disabled="true" />
</type>
```

обязательные параметры:
* type->name — класс, интерфейс или виртуальный тип, с которым будет работать плагин
* plugin->name — произвольное имя плагина, используется для идентификации
* plagin->type — имя класса или виртуального типа, реализующего плагин
опциональные:
* plugin->sortOrder — определяет порядок вызова плагина для плагинов такого же метода
* plagin->disabled — флаг отключения плагина

## Before

Before позволяет изменять аргументы переданные целевой функции, для определения нужно указать в плагине целевой метод с префиксом "before":
```php
public function beforeSetName(\Magento\Catalog\Model\Product $subject, $name)
{
    return ["({$name})"];
}
```

## After

After позволяет изменить значение, котрое возвращает целевой метод, для определения нужно указать в плагине целевой метод с префиксом "after":
```php
public function afterSetName(\Magento\Catalog\Model\Product $subject, $result, $name)
{
    return "|{$result}|";
}
```

в after так-же доступен оригинальный передаваемый аргумент.

## Around

Around позволяет взаимодействовать с целевым методом до и после его исполнения, для определения нужно указать в плагине целевой метод с префиксом "around":
```php
public function afterSave(\Magento\Catalog\Model\Product $subject, \Closure $proceed)
{
    $this->doSmthBeforeProductIsSaved();
    $returnValue = $proceed();
    if ($returnValue) {
        $this->postProductToFacebook();
    }
    return $returnValue;
}
```

В around так-же доступны передаваемые целевому методу аргументы, при вызове `$proceed()` вызываются так-же другие плагины для целевого метода.

## Порядок вызова 

Плагины вызываются в соответствии с `sortOrder`, когда для одного метода определено несколько методов плагина одного типа.

Порядок запуска:
* Before, запускается с наименьшим `sortOrder`
 * Запускаются каждый плагин с методом before
 * Запускается часть around до callable
* После выполянения запускается с наибольшим `sortOrder`
 * Запусается вторая часть around, после callable
 * выполняются after методы

#### Пример

||PluginA|PluginB|PluginC|Action|
|-|-|-|-|-|
|**sortOrder**|10|20|30||
|**before**|beforeDispatch()|beforeDispatch()|beforeDispatch()||
|**around(до callable)**||aroundDispatch() [first half]|aroundDispatch() [first half]||
|**оригнальный**||||dispatch()|
|**around(после callable)**||aroundDispatch() [second half]|aroundDispatch() [second half]||
|**after**|afterDispatch()|afterDispatch()|afterDispatch()||

Будет запущено в следующем порядке:
* PluginA::beforeDispatch()
* PluginB::beforeDispatch()
* PluginB::aroundDispatch() (Magento calls the first half until callable)
  * PluginC::beforeDispatch()
  * PluginC::aroundDispatch() (Magento calls the first half until callable)
    * Action::dispatch()
  * PluginC::aroundDispatch() (Magento calls the second half after callable)
  * PluginC::afterDispatch()
* PluginB::aroundDispatch() (Magento calls the second half after callable)
* PluginB::afterDispatch()
* PluginA::afterDispatch()

## Interception

Interception — это автоматически-генерируемые классы, которые создаются для встраивания плагинов, они наследуются от целевого оригинального класса и обрачивают его методы для вызова плагинов.
