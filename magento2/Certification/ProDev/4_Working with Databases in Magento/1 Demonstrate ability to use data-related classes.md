# Demonstrate ability to use data-related classes

>Describe repositories and data API classes. How do you obtain an object or set of objects from the database using a repository? How do you configure and create a SearchCriteria instance using the builder? How do you use Data/Api classes?

>Describe how to create and register new entities. How do you add a new table to the database?

>Describe the entity load and save process.

>Describe how to extend existing entities. What mechanisms are available to extend existing classes, for example by adding a new attribute, a new field in the database or a new related entity?

>Describe how to filter, sort, and specify the selected values for collections and repositories. How do you select a subset of records from the database?

>Describe the database abstraction layer for Magento. What type of exceptions does the database layer throw? What additional functionality does Magento provide over `Zend_Adapter`?

[How to Use Data-related Classes, Repositories and Data API in Magento 2 (part 1)](https://belvg.com/blog/how-to-use-data-related-classes-repositories-and-data-api-in-magento-2.html)
[Magento 2: Understanding Object Repositories](https://alanstorm.com/magento_2_understanding_object_repositories/)


В M2 для работы с данными сущнотей (CRUD) используются репозитории. Пример репозитория: [Magento\Customer\Model\ResourceModel\CustomerRepository](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Customer/Model/ResourceModel/CustomerRepository.php)

Репозитории представляет из себя класс реализующий интерфейс репозитория, интерфейсы хранятся в директории _Api_ модуля, а реализации в _Model/ResourceModel/_ или _Model_.

Как правило репозитории содержат стандартные методы для CRUD:
```php
namespace Magento\Catalog\Api;
interface ProductRepositoryInterface
{
   public function save(\Magento\Catalog\Api\Data\ProductInterface $product, $saveOptions = false);
   public function get($sku, $editMode = false, $storeId = null, $forceReload = false);
   public function getById($productId, $editMode = false, $storeId = null, $forceReload = false);
   public function delete(\Magento\Catalog\Api\Data\ProductInterface $product);
   public function deleteById($sku);
   public function getList(\Magento\Framework\Api\SearchCriteriaInterface $searchCriteria);
}
```

## Фильтрация данных

[Searching with Repositories](https://devdocs.magento.com/guides/v2.4/extension-dev-guide/searching-with-repositories.html)

Для фильтрации данных используется [`Magento\Framework\Api\SearchCriteriaInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SearchCriteriaInterface.php)

### Фильтры

Для того, чтобы задать условия фильтрации используется класс [`Magento\Framework\Api\Filter`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/Filter.php)

```php
$filter
    ->setField("url")
    ->setValue("%magento.com")
    ->setConditionType("like");
```

### Группы фильтров

Фильтры объеденяются в [`Magento\Framework\Api\Search\FilterGroup`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/Search/FilterGroup.php) затем они предаются в SearchCriteria:

```php
$filter1
    ->setField("url")
    ->setValue("%magento.com")
    ->setConditionType("like");

$filter2
    ->setField("store_id")
    ->setValue("1")
    ->setConditionType("eq");

$filterGroup1->setFilters([$filter1, $filter2]);

$filter3
    ->setField("url_type")
    ->setValue(1)
    ->setConditionType("eq");

$filterGroup2->setFilters([$filter3]);

$searchCriteria->setFilterGroups([$filterGroup1, $filterGroup2]);
```

Между группа используется условие **AND**, между фильтрами одной группы **OR**

### Сортировка

Для сортировки испольуется [`Magento\Framework\Api\SortOrder`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SortOrder.php):

```php
    ->setField("email")
    ->setDirection("ASC");

$searchCriteria->setSortOrders([$sortOrder]);
```

### Пагинация

Пагинация задаётся у SearchCriteria:

```php
$searchCriteria->setPageSize(20); //retrieve 20 or less entities

$searchCriteria
    ->setPageSize(20)
    ->setCurrentPage(2); //show the 21st to 40th entity
```

### Search Result

В результате фильтрации getList возращает объект [`Magento\Framework\Api\SearchResultsInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SearchResultsInterface.php)

### CollectionProcessorInterface

Для унификации примения фильтров, сортировок и пагинаций используется [`Magento\Framework\Api\SearchCriteria\CollectionProcessorInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SearchCriteria/CollectionProcessorInterface.php)

#### Filter Processor

Для примения фильтров используется [`Magento\Framework\Api\SearchCriteria\CollectionProcessor\FilterProcessor`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SearchCriteria/CollectionProcessor/FilterProcessor.php) он применяет группы фильтров для колекции (в методе [`addFilterGroupToCollection()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SearchCriteria/CollectionProcessor/FilterProcessor.php#L59)).

Для полей можно реализовывать кастомные фильтры, по дефолту вызвется метод [`addFieldToFilter()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Data/Collection/AbstractDb.php#L388).

Для добавления кастомного фильтра в `di.xml` нужно прописать примерно такое:

```xml
<virtualType name="Magento\Customer\Model\Api\SearchCriteria\CollectionProcessor\GroupFilterProcessor"
             type="Magento\Framework\Api\SearchCriteria\CollectionProcessor\FilterProcessor">
    <arguments>
        <argument name="customFilters" xsi:type="array">
            <item name="category_id"xsi:type="object">Magento\Catalog\Model\Api\SearchCriteria\CollectionProcessor\FilterProcessor\ProductCategoryFilter</item>
        </argument>
        <argument name="fieldMapping" xsi:type="array">
            <item name="code" xsi:type="string">main_table.customer_group_code</item>
            <item name="id" xsi:type="string">main_table.customer_group_id</item>
            <item name="tax_class_name" xsi:type="string">tax_class_table.class_name</item>
        </argument>
    </arguments>
</virtualType>
```

Реализация [`Magento\Catalog\Model\Api\SearchCriteria\CollectionProcessor\FilterProcessor\ProductCategoryFilter`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Api/SearchCriteria/CollectionProcessor/FilterProcessor/ProductCategoryFilter.php)

Кастомные фильтры должны реализовывать [`Magento\Framework\Api\SearchCriteria\CollectionProcessor\FilterProcessor\CustomFilterInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SearchCriteria/CollectionProcessor/FilterProcessor/CustomFilterInterface.php)


#### Sorting Processor

[`Magento\Framework\Api\SearchCriteria\CollectionProcessor\SortingProcessor`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SearchCriteria/CollectionProcessor/SortingProcessor.php)

Настроить можно в `di.xml`:

```xml
    <virtualType name="Magento\Customer\Model\Api\SearchCriteria\CollectionProcessor\GroupSortingProcessor" type="Magento\Framework\Api\SearchCriteria\CollectionProcessor\SortingProcessor">
        <arguments>
            <argument name="fieldMapping" xsi:type="array">
                <item name="code" xsi:type="string">customer_group_code</item>
                <item name="id" xsi:type="string">customer_group_id</item>
                <item name="tax_class_name" xsi:type="string">class_name</item>
            </argument>
            <argument name="defaultOrders" xsi:type="array">
                <item name="id" xsi:type="string">ASC</item>
            </argument>
        </arguments>
    </virtualType>
```

#### Pagination Processor

[`Magento\Framework\Api\SearchCriteria\CollectionProcessor\PaginationProcessor`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SearchCriteria/CollectionProcessor/PaginationProcessor.php)

#### Join Processor

[`Magento\Framework\Api\SearchCriteria\CollectionProcessor\JoinProcessor`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SearchCriteria/CollectionProcessor/JoinProcessor.php)

конфиг в `di.xml`:
```xml
<virtualType name="Magento\Tax\Model\Api\SearchCriteria\CollectionProcessor\TaxRuleJoinProcessor" type="Magento\Framework\Api\SearchCriteria\CollectionProcessor\JoinProcessor">
  <arguments>
    <argument name="customJoins" xsi:type="array">
      <item name="rate.tax_calculation_rate_id" xsi:type="object">Magento\Tax\Model\Api\SearchCriteria\JoinProcessor\Rate</item>
      <item name="rc.code" xsi:type="object">Magento\Tax\Model\Api\SearchCriteria\JoinProcessor\RateCode</item>
      <item name="ctc.customer_tax_class_id" xsi:type="object">Magento\Tax\Model\Api\SearchCriteria\JoinProcessor\CustomerTaxClass</item>
      <item name="ptc.product_tax_class_id" xsi:type="object">Magento\Tax\Model\Api\SearchCriteria\JoinProcessor\ProductTaxClass</item>
      <item name="cd.customer_tax_class_id" xsi:type="object">Magento\Tax\Model\Api\SearchCriteria\JoinProcessor\CalculationData</item>
      <item name="cd.product_tax_class_id" xsi:type="object">Magento\Tax\Model\Api\SearchCriteria\JoinProcessor\CalculationData</item>
    </argument>
    <argument name="fieldMapping" xsi:type="array">
      <item name="id" xsi:type="string">tax_calculation_rule_id</item>
      <item name="tax_rate_ids" xsi:type="string">tax_calculation_rate_id</item>
      <item name="customer_tax_class_ids" xsi:type="string">cd.customer_tax_class_id</item>
      <item name="product_tax_class_ids" xsi:type="string">cd.product_tax_class_id</item>
      <item name="tax_calculation_rate_id" xsi:type="string">rate.tax_calculation_rate_id</item>
    </argument>
  </arguments>
</virtualType>
```

## Load/save

`load()` метод в модели в M2 объявлен устаревшим, для загрузки нужно использовать репозитории:
```php
public function getById($blockId)
{
    $block = $this->blockFactory->create();
    $this->resource->load($block, $blockId);
    if (!$block->getId()) {
        throw new NoSuchEntityException(__('CMS Block with id "%1" does not exist.', $blockId));
    }
    return $block;
}
```

`save()` аналогично выполяется посредством репозитория


## Дополнительно

[Magento 2: CRUD Models for Database Access](https://alanstorm.com/magento_2_crud_models_for_database_access/)

## Архитекута дата слоя

### Модели

Модель представляет из себя репрезентацию данных записи из БД. Каждая запись, загруженаая через коллекции или ресурсные модели представляется в модели. 

Модели наследуются от [`Magento\Framework\Model\AbstractModel`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Model/AbstractModel.php).
Модели которые используют extenstion attributes наследуются от [`Magento\Framework\Model\AbstractExtensibleModel`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Model/AbstractExtensibleModel.php).

### Ресурсные модели

Ресурсная модель взаимодействует с БД, грузит данные, сохраняет их, в ней также можно выполнять и кастомные запросы к БД. Ресурсные модели используются в моделях, репозториях и коллекциях.

Базовый класс: [`Magento\Framework\Model\ResourceModel\Db\AbstractDb`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Model/ResourceModel/Db/AbstractDb.php)

### Коллекции

Коллекции предназначены для загрузки и работы с несколькими одинаковыми сущностями загруженными из БД. Коллекции используются в репозиториях и в отличии от них дают более гибкие возможности для работы с данными, например для EAV коллекций можно указывать какие атрибуты нужно загружать, а в репозитории обычно такой возможности нет и грузятся все атрибуты. Коллекции являются более низкоуровнеыми, чем репозтории.

Базовый класс: [`Magento\Framework\Model\ResourceModel\Db\Collection\AbstractCollection`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Model/ResourceModel/Db/Collection/AbstractCollection.php)

### Репозитории

Репозиторий представляет из себя более высокоуровневую комбинацию ресурсных моделей и коллекций. Он позволяет загружать как одиночные модели, так и массивы моделей, отфильтрованные с помощью [`Magento\Framework\Api\SearchCriteriaInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Api/SearchCriteriaInterface.php). 

Репозитрии не имеют базового класса, но для них нужно делать интерфейсы в _`<module-dir>`/Api_. 

Пример репозитория: [`Magento\Cms\Model\BlockRepository`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Cms/Model/BlockRepository.php).