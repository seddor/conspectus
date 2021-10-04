# Demonstrate ability to use EAV entity load and save

>Describe the EAV load and save process and differences from the flat table load and save process. What happens when an EAV entity has too many attributes? How does the number of websites/stores affect the EAV load/save process? How would you customize the load and save process for an EAV entity in the situations described here?

[EAV Load and Save Processes in Magento 2 ](https://belvg.com/blog/eav-load-and-save-processes-in-magento-2.html)

В M2 для процесинга EAV и Flat объектов ипользуется [`Magento\Framework\EntityManager\EntityManager`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/EntityManager.php)

Для работы с EntityManager нужно предоставить информацию в `di.xml`:
```xml
<type name="Magento\Framework\EntityManager\MetadataPool">
   <arguments>
       <argument name="metadata" xsi:type="array">
           <item name="MyVendor\MyModule\Api\Data\MyEntityInterface" xsi:type="array">
               <item name="entityTableName" xsi:type="string">myvendor_mymodule_myentity_entity</item>
               <item name="eavEntityType" xsi:type="string">myvendor_mymodule_myentity</item>
               <item name="identifierField" xsi:type="string">entity_id</item>
               <item name="entityContext" xsi:type="array">
                   <item name="store" xsi:type="string">Magento\Store\Model\StoreScopeProvider</item>
               </item>
           </item>
       </argument>
   </arguments>
</type>

<type name="Magento\Framework\EntityManager\HydratorPool">
   <arguments>
       <argument name="hydrators" xsi:type="array">
           <item name="MyVendor\MyModule\Api\Data\MyEntityInterface" xsi:type="string">Magento\Framework\EntityManager\AbstractModelHydrator</item>
       </argument>
   </arguments>
</type>
```

[`Magento\Framework\EntityManager\OperationPool`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/OperationPool.php) содержит массив операций, которые EntityManager использует для работы с объектом.

Дефолтные операции:

* checkIfExists — [`Magento\Framework\EntityManager\Operation\CheckIfExists`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/CheckIfExists.php) — SQL запросом проверяет наличие в БД.
* read — [`Magento\Framework\EntityManager\Operation\Read`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Read.php) — содержит подоперации:
    * [`ReadMain`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Read/ReadMain.php)
    * [`ReadAttributes`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Read/ReadAttributes.php)
    * [`ReadExtensions`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Read/ReadExtensions.php)
* create — [`Magento\Framework\EntityManager\Operation\Create`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Create.php) — содержит подоперации:
    * [`CreateMain`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Create/CreateMain.php)
    * [`CreateAttributes`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Create/CreateAttributes.php)
    * [`CreateExtensions`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Create/CreateExtensions.php)
* update — [`Magento\Framework\EntityManager\Operation\Update`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Update.php) — содержит подоперации:
    * [`CreateMain`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Create/CreateMain.php)
    * [`CreateAttributes`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Create/CreateAttributes.php)
    * [`CreateExtensions`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Create/CreateExtensions.php)
* delete — [`Magento\Framework\EntityManager\Operation\Delete`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Delete.php) — содержит подоперации:
    * [`DeleteMain`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Delete/DeleteMain.php)
    * [`DeleteAttributes`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Delete/DeleteAttributes.php)
    * [`DeleteExtensions`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/Delete/DeleteExtensions.php)

Операции для работы с атрибутами собираются в [`Magento\Framework\EntityManager\Operation\AttributePool`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/EntityManager/Operation/AttributePool.php)

Для атрибутов EAV использует хендлы (записываются в `AttributePool`) расщиряющие операции:

* [`Magento\Eav\Model\ResourceModel\CreateHandler`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/ResourceModel/CreateHandler.php)
* [`Magento\Eav\Model\ResourceModel\UpdateHandler`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/ResourceModel/UpdateHandler.php)
* [`Magento\Eav\Model\ResourceModel\ReadHandler`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Eav/Model/ResourceModel/ReadHandler.php)

## Влияние количества сайтов/сторов на процесинг EAV

Работа с разделением значений по сторам просходит так:

* в таблицах значений есть колонка `store_id`, если она равна:
    * 0 — это глобальное/дефолтное значение для всех сторов
    * ID стора — это специфичное значение для соответствующего стора

Загрзука просходит так: добавляется филтьр store_id IN (STORE_IDS), значение берётся в зависимости от текущего стора, если такого нет, то берётся от стора фалбека, если нет то 0.

В большинстве случаев для предоставляения значений сторов используется [`Magento\Store\Model\StoreScopeProvide`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Store/Model/StoreScopeProvider.php) который использует текущий store ID и если он не равен 0, то добавляет в фалбек 0.

Таким образом в большинстве случаев на скорость загрузки количество сторов не влияет. 

Повлиять на скорость сохранение может наличие атрибутов со скопом `website`, т.к в этом случае значение аттрибутов нужно сохранять для каждого стор вью вебсайта.

## Кастомизация процесинга EAV

Для переопределения операций, нужно добавить в `di.xml`:
```xml
<type name="Magento\Framework\EntityManager\OperationPool">
   <arguments>
       <argument name="operations" xsi:type="array">
           <item name="MyVendor\MyModule\Api\Data\MyEntityInterface" xsi:type="array">
               <item name="checkIsExists" xsi:type="string">MY_NAMESPACE\CheckIsExists</item>
               <item name="read" xsi:type="string">MY_NAMESPACE\Read</item>
               <item name="create" xsi:type="string">MY_NAMESPACE\Create</item>
               <item name="update" xsi:type="string">MY_NAMESPACE\Update</item>
               <item name="delete" xsi:type="string">MY_NAMESPACE\Delete</item>
           </item>
       </argument>
   </arguments>
</type>
```
Для переопределения операций работы с EAV атрибутами, нужно добавить в `di.xml`:
```xml
<type name="Magento\Framework\EntityManager\Operation\AttributePool">
   <arguments>
       <argument name="extensionActions" xsi:type="array">
           <item name="eav" xsi:type="array">
               <item name="MyVendor\MyModule\Api\Data\MyEntityInterface" xsi:type="array">
                   <item name="read" xsi:type="string">MY_NAMESPACE\ReadHandler</item>
                   <item name="create" xsi:type="string">MY_NAMESPACE\CreateHandler</item>
                   <item name="update" xsi:type="string">MY_NAMESPACE\UpdateHandler</item>
               </item>
           </item>
       </argument>
   </arguments>
</type>
```
Для расширения операций, нужно добавить в `di.xml`:
```xml
<type name="Magento\Framework\EntityManager\Operation\ExtensionPool">
   <arguments>
       <argument name="extensionActions" xsi:type="array">
           <item name="MyVendor\MyModule\Api\Data\MyEntityInterface" xsi:type="array">
               <item name="read" xsi:type="array">
                   <item name="myReader" xsi:type="string">MY_NAMESPACE\ReadHandler</item>
               </item>
               <item name="create" xsi:type="array">
                   <item name="myCreator" xsi:type="string">MY_NAMESPACE\CreateHandler</item>
               </item>
               <item name="update" xsi:type="array">
                   <item name="myUpdater" xsi:type="string">MY_NAMESPACE\UpdateHandler</item>
               </item>
           </item>
       </argument>
   </arguments>
</type>
```
