# API фреймворк

Operational API можно разделить на две части:

* Buisness logic API — отвечает за бизнесс операции
* Repositories — предоставляют эквивален коллекций для сервисного уровня

## Buisness logic API

Примером является [`Magento\Catalog\Api\ProductTypeListInterface`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Catalog/Api/ProductTypeListInterface.php):
```php
/**
 * @api
 * @since 100.0.2
 */
interface ProductTypeListInterface
{
    /**
     * Retrieve available product types
     *
     * @return \Magento\Catalog\Api\Data\ProductTypeInterface[]
     */
    public function getProductTypes();
}
```

Пример имелементации [`Magento\Customer\Api\AccountManagementInterface::authenticate()`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Customer/Api/AccountManagementInterface.php#L111) в [`Magento\Customer\Model\AccountManagement::authenticate()`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Customer/Model/AccountManagement.php#L594).

Как првило не используют и не наследуют классов фреймворка.

## Repositories

Пример интерфейса [`Magento\Catalog\Api\ProductRepositoryInterface`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Catalog/Api/ProductRepositoryInterface.php)

Пимер реализации [`Magento\Catalog\Model\ProductRepository`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Catalog/Model/ProductRepository.php)

### Типичный сценарий

Как правило репозиторий это интерфейс, который обеспечивает доступ к наборы объектов из метода `getList()`. Он не обязан использовать что либо из фреймворка, однако как правило метод принимает как аргумет [`Magento\Framework\Api\SearchCriteriaInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Api/SearchCriteriaInterface.php), который является частью фреймворка, так-же фреймворк предоставляет реазлиацию интерфейса в [` Magento\Framework\Api\SearchCriteria`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Api/SearchCriteria.php).

Как правило репозитории не наследуются от классов фремворка, но обычно используют `SearchCriteria`.

## Data API

Обычно может наследовать [`Magento\Framework\Api\AbstractExtensibleObject`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Api/AbstractExtensibleObject.php) или [`Magento\Framework\Api\AbstractSimpleObject`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Api/AbstractSimpleObject.php), или ничего.
