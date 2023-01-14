# Воркфлоу моделей

## Чтение

Загрузка моделей происохдит с помощью ресурсной модели, метода [`load()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/ResourceModel/Db/AbstractDb.php#L344)

load(\Magento\Framework\Model\AbstractModel $object, $value, $field = null), здесь:

* `$object` — модель которую нужно загрузить
* `$value` — значение, по которому происходит поиск записи в БД
* `$field` — поле по котрому ищутся значения, если не указан используется поле указанное как идентификатор в модели

В методе [`_getLoadSelect()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/ResourceModel/Db/AbstractDb.php#L379) формируется SQL-запрос для загрузки модели.

### $model->__beforeLoad()

Пперед загрузкой у модели вызвается метод [`_beforeLoad()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L564), он вызывает два события:

* `model_load_before`
* `<$this->_eventPrefix>_load_before`

Эти события могут использоватся для проверок бизенс уровня или ACL, чтобы поверить можно ли загружать эту модель из БД.

Этот метод является устаревшим и эти события использовать не рекомендуется, вместо них можно сделать плагны на `load()` ресурсной модели.


### $model->__afterLoad()

После загрузки у модели вызывается метод [`_afterLoad()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L578), он вызывает два события:

* `model_load_after`
* `<$this->_eventPrefix>_load_after`

### События в коллекциях

При загрузке коллекции события `model_load_before` и `model_load_after` не вызываются, вместо этого в коллекциях можно использовать:

* `core_collection_abstract_load_before`
* `core_collection_abstract_load_after`

### Исходные данные в модели

После загрзуки модели вызывается метод [`updateStoredData()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L965), которых сохраняет в поле `storedData` оригинальные данные модели, которые были получены из БД. Затем сохранёные данные можно получить методом [`getStoredData()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L980). 

### Флаг изменения

С помощью метода [`hasDataChanges()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L331) можно проверить изменилась ли модель с момента загрузки/сохранения. Метод возвращает значение флага `_hasDataChanges`. Флаг изменяется при вызовах методов `setData()`, `addData()` и магических сетеров.

## Создание и обновление

Сохранение происходит с помощью метода ресурсной модели [`save()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/ResourceModel/Db/AbstractDb.php#L396) в метод передаётся объект который нжуно сохранить.

В модели метод `save()` является устаревшим.

### Флаг удаление

Метод модели [`isDeleted()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L314) возвращает текущее значение флага `_isDeleted`, в аргументе можно передать нужное значение флага. Этот флаг обозначает, что объект отмечен для удаления, используется для массового обновления записей через коллекцию.

### Валидация

Перед сохранением для валидации вызывается метод модели [`validateBeforeSave()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L712).

### beforeSave

Перед сохранением вызвается метод модели [`beforeSave()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L696), он взывает события:

* `model_save_before`
* `<$this->_eventPrefix>_save_before`

События используются для проверки на уровне бизнес логики или ACL.

### isSaveAllowed

Перед сохранением вызвается метод модели [`isSaveAllowed()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L627), возвращающий значение флага `_dataSaveAllowed`. Если флаг false то фактического сохранения данных в БД не будет.

### afterSave

После сохранения вызвается метод модели [`afterSave()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L824), он вызывает события:

* `model_save_after`
* `clean_cache_by_tags`
* `<$this->_eventPrefix>_save_after`

Так-же в конце `afterSave()` вызвается метод `updateStoredData()`, заменяющий исходные данные в модели.

### afterCommitCallback

После удачного сохранения вызвается метод моедли [`afterCommitCallback()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L664), вызывает события:

* `model_save_commit_after`
* `<$this->_eventPrefix>_save_commit_after`

## Удаление

Для удаление используется метод [`delete()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/ResourceModel/Db/AbstractDb.php#L447) ресурсной модели, в него передаёся удаляемый объект.

В модели метод `delete()` является устаревшим.

### beforeDelete

Перед удалением вызвается метод модели [`beforeDelete()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L855), вызывающий события:

* `model_delete_before`
* `<$this->_eventPrefix>_delete_before`

### afterDelete

После удаления вызвается метод модели [`afterDelete()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L874), он вызывает события:

* `model_delete_after`
* `clean_cache_by_tags`
* `<$this->_eventPrefix>_delete_after`

### afterDeleteCommit

После завершения транзакции удаления вызвается метод модели [`afterDeleteCommit()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Model/AbstractModel.php#L888), он вызывает события:

* `model_delete_commit_after`
* `<$this->_eventPrefix>_delete_commit_after`
