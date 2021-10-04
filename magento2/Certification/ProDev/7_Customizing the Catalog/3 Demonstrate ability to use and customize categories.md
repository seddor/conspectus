# Demonstrate ability to use and customize categories

>Describe category properties and features. How do you create and manage categories?
>Describe the category hierarchy tree structure implementation (the internal structure inside the database). What is the meaning of parent_id 0? How are paths constructed? Which attribute values are required to display a new category in the store? What kind of strategies can you suggest for organizing products into categories?

[Demonstrating the Ability to Use and Customize Categories in Magento 2](https://belvg.com/blog/demonstrating-the-ability-to-use-and-customize-categories-in-magento-2.html)

## Describe category properties and features

Фичи категорий:

* Строятся в иерархическую, древовидную структуру. Эта структура используется в меню сайта
* Категория может быть якорной (anchored), в якорной категории будут отображаться продукты из дочерних категорий, без прямого включения продуктов в якорную
* Категории имеют разные типы отображения:
  * только продукты
  * только CMS-блок
  * продуты и CMS-блок
* В категоии можно назначать порядок сортровки для связянных с ней продуктов. Для якорных категоий продукты назначенные напрямую в категории будт всегда выше продуктов из дочерних категорий.
* Категории можно назначить кастомный дизайн (тему, обнолвение лаяуту), также можно ограничить прменения кастомного дизайна во времени

### How do you create and manage categories?

Категории можно создать в админке в _Catalog — Category_ или программно.

Для сохранения модели категории используется репозиторий категорий [`Magento\Catalog\Model\CategoryRepository`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/CategoryRepository.php), реализующий [`Magento\Catalog\Api\CategoryRepositoryInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Api/CategoryRepositoryInterface.php) у него есть метод [`save()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/CategoryRepository.php#L77). 

Для манеджмента категорий можно использовать класс [`Magento\Catalog\Model\CategoryManagement`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/CategoryManagement.php), реализующий [`Magento\Catalog\Api\CategoryManagementInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Api/CategoryManagementInterface.php), у него есть методы:
  * [`getTree()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/CategoryManagement.php#L59) — позволяет получить дерево категорий, возвращает [`Magento\Catalog\Api\Data\CategoryTreeInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Api/Data/CategoryTreeInterface.php)
  * [`move()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/CategoryManagement.php#L119) — передвигает категорию в иерархии.
  * [`getCount()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/CategoryManagement.php#L148) — возвращает общее количество категорий.

Модель категоии: [`Magento\Catalog\Model\Category`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php), реализует интерфейсы:
* [`Magento\Framework\DataObject\IdentityInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/DataObject/IdentityInterface.php)
* [`Magento\Catalog\Api\Data\CategoryInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Api/Data/CategoryInterface.php)
* [`Magento\Catalog\Api\Data\CategoryTreeInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Api/Data/CategoryTreeInterface.php)
методы:
* [`getProductCollection()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L474) — взовращает продуктовую коллекцию с фильтром по категории и стору
* [`getProductsPosition()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L515) — возвращает массив с назначеными для продуктов позициями
* [`getUrl()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L608) — URL категории
* [`getImageUrl()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L671) — URL изображения категории, в параметре указывает атрибут изображения
* [`getParentCategory()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L707) — родительская категория
* [`getParentIds()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L735) — все ИД родительских категорий
* [`getAllChildren()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L790) — возвращает список ИД всех активных дочерних категорий, по умолчанию возвращает строку с разделителем ",", или в виде массива, если передать параметр.
* [`getChildren()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L808) — возвращает список ИД всех активных активных непосредственно дочерних категорий, возвращает строку с разделителем ",".
* [`hasChildren()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L890) — проверяет по всем уровням, только активные категоии.
* [`getAnchorsAbove()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L934) — возвращает массив ИД всех родительских якорных категорий.
* [`getProductCount()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L964) — возвращает список ИД всех активных дочерних категорий, по умолчанию возвращает строку с разделителем ",", или в виде массива, если передать параметр.
* [`getCategories()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L984) — коллекция категорий дочерних к ИД переданому в первом параметре, только активные, включенные в меню также к колеекции джойнятся реврайты урлов, возвращает [`Magento\Framework\Data\Tree\Node\Collection`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Data/Tree/Node/Collection.php) или [`Magento\Catalog\Model\ResourceModel\Category\Collection`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/ResourceModel/Category/Collection.php)
* [`getParentCategories()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L995) — массив родительских категорий, только активные
* [`getChildrenCategories()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/Category.php#L1005) — массив непосредственных дочерних категорий, только активные, включенные в меню, с реврайтами, вовзращает [`Magento\Catalog\Model\ResourceModel\Category\Collection`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/ResourceModel/Category/Collection.php)

Ресурсная модель модели категории в зависимости от включения flat для категоий:
* [`Magento\Catalog\Model\ResourceModel\Category`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/ResourceModel/Category.php) — без включаения flat
* [`Magento\Catalog\Model\ResourceModel\Category\Flat`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Catalog/Model/ResourceModel/Category/Flat.php) — с включенным flat

## Describe the category hierarchy tree structure implementation

Каждая категория имеет поля иерархии: `parent_id`, `level`, `path` эти поля хранятся в основной таблице сущности `catalog_category_entity`.

* `parent_id` — ИД непосредственного родителя категории
* `level` — отражает уровень вложености в иерархии, соответственно 0 — начало (вершина).
* `path` — путь категории


| entity_id | parent_id | path         | level |
|-----------|-----------|--------------|-------|
|         1 |         0 | 1            |     0 |
|         2 |         1 | 1/2          |     1 |
|         3 |         2 | 1/2/3        |     2 |
|         4 |         3 | 1/2/3/4      |     3 |
|         5 |         3 | 1/2/3/5      |     3 |
|         6 |         3 | 1/2/3/6      |     3 |
|         7 |         2 | 1/2/7        |     2 |
|         8 |         7 | 1/2/7/8      |     3 |

### What is the meaning of parent_id 0?

Начальная категория находящаяся в самом верху иерархии имеет parent_id = 0, такая категория может быть только одна. Все категории начинающее дерево категорий назначаются дочерними категориями этой категоии.

### How are paths constructed?

`path` строится как разделённый "/" рекурсивный список непосредственных родительских категорий от текущей, до самой верхней (с parent_id = 0) с включением ИД текущей категории.

Например для категории с ИД 3 и родителем с ИД 2, path будет: _1/2/3_

### Which attribute values are required to display a new category in the store? 

При создании категории в админке обязательно указать `name` категории, также для отображения категории нужно:

* `is_active` — 1
* `include_in_menu` — 1, если категория должна быть в меню, в не зависмости от значения категория будет доступна по прямой ссылке
* `url_key` — генерируется автоматически при сохранении на основе `name`

### What kind of strategies can you suggest for organizing products into categories?

Чаще всего способ организации товаров в категории зависит от бизнес задач.

Один из вариантов разделения продуктов по категориям, если на сайте используются разные наборы атрибутов для товаров, то можно делить товары с разными наборами в разные категории. Тогда на каждой такой категории для продуктов будут предоставлятся наиболее релевантные для этого набора атрибутов фильтры.

## Атрибуты

[Add a category attribute](https://devdocs.magento.com/guides/v2.4/ui_comp_guide/howto/add_category_attribute.html)
