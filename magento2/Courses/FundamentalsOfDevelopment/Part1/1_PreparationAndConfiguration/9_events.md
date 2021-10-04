# События

События обычно используются для обработки внешних действий или ввода. Событие часть патерна Event-Observer.

## Пример

1. Класс `Magento\Checkout\Model\Type\Onepage`, метод `saveOrder()` вызывает событие:
```php
$this->_eventManager->dispatch(
    'checkout_submit_all_after',
    [
        'order' => $order,
        'quote' -> $this->getQuote(),
    ]
);
```
2. Конфиг `Magento\CatalogInventory\etc\events.xml` содержит определение обработчика:
```xml
<event name="checkout_submit_all_after">
    <observer name="inventory" instance="Magento\CatalogInventory\Observer\CheckoutAllSubmitAfterObserver">
</event>
```
3. Класс обработчика `Magento\CatalogInventory\Observer\CheckoutAllSubmitAfterObserver` обрабатывает событие в методе `execute()`:
```php
namespace Magento\CatalogInventory\Observer;

use Magento\Framework\Event\ObserverInterface;
use Magento\Framework\Event\Observer;

class CheckoutAllSubmitAfterObserver extends ObserverInterface
{
    public function execute(Observer $observer)
    {
        $quote = $observer->getEvent()->getQuote();
        if (!$quote->getInventoryProcessed()) {
            $this->subtractQuoteInventoryObserver->execute($observer);
            $this->reindexQuoteInventoryObserver->execute($observer);
        }
        return $this;
    }
}
```

В M2, в отличии от M1 один обработчик обрабатывает только одно событие, каждый обработчик реализует интерфейс [`Magento\Framework\Event\ObserverInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Event/ObserverInterface.php), который содержит один метод `execute(Observer $observer)`.
