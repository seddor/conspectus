# Determine and manage catalog rules

>Identify how to implement catalog price rules. When would you use catalog price rules? How do they impact performance? How would you debug problems with catalog price rules?

Логика каталожный правил реализована в модуле [`Magento_CatalogRule`](https://github.com/magento/magento2/tree/2.4/app/code/Magento/CatalogRule)

## Структура БД

Структура каталожных правил в БД состоит из следующих таблиц:

Основные таблицы:
* `catalogrule` — основная информация о правилах
* `catalogrule_website` — соотношения правил и вебсайтов
* `catalogrule_customer_group` — соотношения правил и групп кастомеров
Таблицы индексов:
* `catalogrule_product` — содержит продукты, группу кастомера, время начало и конца действия, тип и размер скидки
* `catalogrule_product_price` — содержит продукты, группу кастомера, дату правила, цену правила, latest_start_date, earlier_end_date
* `catalogrule_group_website` — правило и его группа кастомера, вебсайт. Содержит только действующие сегодня соотношения (по таблице `catalogrule_product`)
Реплики — используются для быстрой смены индексированных данных:
* `catalogrule_product_replica`
* `catalogrule_product_price_replica`
* `catalogrule_group_website_replica`

## Применение правил

Автоматическое применение правил происходит так:

1. В `0 1 * * *` стартует крон джоба `catalogrule_apply_all`, запускающая [`Magento\CatalogRule\Cron\DailyCatalogUpdate::execute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Cron/DailyCatalogUpdate.php):
```php
public function execute()
{
    $this->ruleProductProcessor->markIndexerAsInvalid();
}
```

Нажатие кнопки ["применить всё"](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Rule/Job.php#L57) и ["сохранить и применить"](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Controller/Adminhtml/Promo/Catalog/Save.php#L126) в админке по сути делает тоже самое. 

2. Класс [`Magento\CatalogRule\Model\Indexer\Rule\RuleProductProcessor`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/Rule/RuleProductProcessor.php) наследует [`Magento\Framework\Indexer\AbstractProcessor`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Indexer/AbstractProcessor.php), сам класс является пустым и только переопределяет константу `INDEXER_ID`:
```php
const INDEXER_ID = 'catalogrule_rule';
```
метод [`markIndexerAsInvalid()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Indexer/AbstractProcessor.php#L87) берётся из родительского класса:
```php
public function markIndexerAsInvalid()
{
    $this->getIndexer()->invalidate();
}
```
3. [`getIndexer()`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Indexer/AbstractProcessor.php#L37) инициирует процесс загрузки модели индекса
```php
/**
 * Get indexer
 *
 * @return \Magento\Framework\Indexer\IndexerInterface
 */
public function getIndexer()
{
    return $this->indexerRegistry->get(static::INDEXER_ID);
}
```
indexerRegistry это [`Magento\Framework\Indexer\IndexerRegistry`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Indexer/IndexerRegistry.php), в методе `get()` он создаёт экземпляр [`Magento\Framework\Indexer\IndexerInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Indexer/IndexerInterface.php) который загружает модель нужного индекса, затем эта модель помещается в поле `indexers` внутри IndexerRegistry и в дальнейшем получается из него.
```php
/**
 * Retrieve indexer instance by id
 *
 * @param string $indexerId
 * @return IndexerInterface
 */
public function get($indexerId)
{
    if (!isset($this->indexers[$indexerId])) {
        $this->indexers[$indexerId] = $this->objectManager->create(
            \Magento\Framework\Indexer\IndexerInterface::class
        )->load($indexerId);
    }
    return $this->indexers[$indexerId];
}
```
4. `IndexerInterface` реализует класс [`Magento\Indexer\Model\Indexer`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Indexer/Model/Indexer.php). **Следует обратить внимание, что `IndexerInterface` является устаревшим, для новых индексов нужно использать [`Magento\Framework\Indexer\ActionInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Indexer/ActionInterface.php)**.
Для загрузки модели индекса у `Indexer` используется метод [`load()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Indexer/Model/Indexer.php#L220), он достаёт данные индекса из конфига и сетает их в текущую модель индекса:
```php
public function load($indexerId)
{
    $indexer = $this->config->getIndexer($indexerId);
    if (empty($indexer) || empty($indexer['indexer_id']) || $indexer['indexer_id'] != $indexerId) {
        throw new \InvalidArgumentException("{$indexerId} indexer does not exist.");
    }

    $this->setId($indexerId);
    $this->setData($indexer);

    return $this;
}
```
5. После загрзуки индекса вызывается метод [`invalidate()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Indexer/Model/Indexer.php#L333), вызыванный на шаге 2.
```php
public function invalidate()
{
    $state = $this->getState();
    $state->setStatus(StateInterface::STATUS_INVALID);
    $state->save();
}
```
6. Каждую минуту запускается крон джоба `indexer_reindex_all_invalid`, которая вызывает индексацию всех невалидных индексов. Она запускает [`Magento\Indexer\Cron\ReindexAllInvalid::execute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/Indexer/Cron/ReindexAllInvalid.php#L29):
```php
$this->processor->reindexAllInvalid();
```
7. Запускается индексирование `catalogrule_rule`, класс индекса: [`Magento\CatalogRule\Model\Indexer\Rule\RuleProductIndexer`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/Rule/RuleProductIndexer.php), он наследуется от класса [`Magento\CatalogRule\Model\Indexer\AbstractIndexer`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/AbstractIndexer.php)
8. Полное индексирование выполняется в методе [`executeFull()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/AbstractIndexer.php#L68):
```php
$this->indexBuilder->reindexFull();
$this->_eventManager->dispatch('clean_cache_by_tags', ['object' => $this]);
$this->getCacheManager()->clean($this->getIdentities());
```
9. `indexBuilder` это класс [`Magento\CatalogRule\Model\Indexer\IndexBuilder`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/IndexBuilder.php), метод [`reindexFull()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/IndexBuilder.php#L315):
```php
public function reindexFull()
{
    try {
        $this->doReindexFull();
    } catch (\Exception $e) {
        $this->critical($e);
        throw new \Magento\Framework\Exception\LocalizedException(
            __("Catalog rule indexing failed. See details in exception log.")
        );
    }
}
```
метод [`doReindexFull()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/IndexBuilder.php#L332):
```php
protected function doReindexFull()
{
    foreach ($this->getAllRules() as $rule) {
        $this->reindexRuleProduct->execute($rule, $this->batchCount, true);
    }

    $this->reindexRuleProductPrice->execute($this->batchCount, null, true);
    $this->reindexRuleGroupWebsite->execute(true);

    $this->tableSwapper->swapIndexTables(
        [
            $this->getTable('catalogrule_product'),
            $this->getTable('catalogrule_product_price'),
            $this->getTable('catalogrule_group_website')
        ]
    );
}
```
10. `reindexRuleProduct` это класс [`Magento\CatalogRule\Model\Indexer\ReindexRuleProduct`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/ReindexRuleProduct.php), метод [`execute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/ReindexRuleProduct.php#L69) вызывается для каждого правила, выполнение происходит только для активных правил у которых указаны вебсайты для применения. Если правило применимо, то для его продуктов, полученных в методе [`Magento\CatalogRule\Model\Rule::getMatchingProductIds()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Rule.php#L342), готовятся данные и заносятся во временную таблицу для `catalogrule_product`.
11. `reindexRuleProductPrice` это класс [`Magento\CatalogRule\Model\Indexer\ReindexRuleProductPrice`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/ReindexRuleProductPrice.php), метод [`execute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/ReindexRuleProductPrice.php#L72) готовит данные по ценам продуктов на сегодняшний, прошедший и завтрашний день. Затем вызывает [`Magento\CatalogRule\Model\Indexer\RuleProductPricesPersistor::execute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/RuleProductPricesPersistor.php#L65) которые заполняет временую таблицу для `catalogrule_product_price`.
12. `reindexRuleGroupWebsite` это класс [`Magento\CatalogRule\Model\Indexer\ReindexRuleGroupWebsite`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/ReindexRuleGroupWebsite.php), метод [`execute()`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/ReindexRuleGroupWebsite.php#L68) заполняет соотношения групп кастомеров и вебсайтов к действующим правилам (для правил у которых есть продукты в `catalogrule_product`) и заполняет их во времнную таблицу для `catalogrule_group_website`.
13. `tableSwapper` это класс [`Magento\CatalogRule\Model\Indexer\IndexerTableSwapper`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/ReindexRuleProductPrice.php) реализующий [`Magento\CatalogRule\Model\Indexer\IndexerTableSwapperInterface`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/IndexerTableSwapperInterface.php), метод [`swapIndexTables`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Model/Indexer/IndexerTableSwapper.php#L87) выполняет замену временных и постоянных таблиц, затем удаляет все временные.

## Identify how to implement catalog price rules.

Каталожная цена реализована в классе [`Magento\CatalogRule\Pricing\Price\CatalogRulePrice`](https://github.com/magento/magento2/blob/2.4/app/code/Magento/CatalogRule/Pricing/Price/CatalogRulePrice.php) он наследует [`Magento\Framework\Pricing\Price\AbstractPrice`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Price/AbstractPrice.php) и реализует [`Magento\Framework\Pricing\Price\BasePriceProviderInterface`](https://github.com/magento/magento2/blob/2.4/lib/internal/Magento/Framework/Pricing/Price/BasePriceProviderInterface.php). По скольку модель реализует `BasePriceProviderInterface`, то она учавствует в получении финальной цены продукта.

### When would you use catalog price rules?

Каталожные правила используются для предоставление скидки на продукт до добавления его в корзину. Каталожные правила можно применять, например, для таких кейсов:

* Скидка для какой-нибудь категории (или набору продуктов по признакам) на определённый период
* Скидка группе клиентов (например для зарегистрированных)

### How do they impact performance?

* Добавлет SQL запрос во время получения финальной цены

### How would you debug problems with catalog price rules?

1. Проверить наличие продукта в `catalogrule_product`
2. Провеить правило, что его не прекрывают какие-нибудь другие правила и что оно применимо
3. Проверить цену продукта в `catalogrule_product_price`
