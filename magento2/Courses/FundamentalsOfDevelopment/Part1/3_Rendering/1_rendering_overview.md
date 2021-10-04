# Templates (шаблоны)

Темплейты представляют из себя `.phtml` файлы содержащие HTML со вставками PHP.

В M2 возможно использовать любую систему рендеринга, помимо phtml, в отличии от M1, где это было единственным вариантом.

# Блоки

Блоки это PHP-классы, которые связаны с темплейтом.

# UiComponents

UiComponents рендарятся при помощи JS, бекэнд в них используется только для получения данных.

# Design layout

Лаяут — xml-файл определяющий структуру страницы. 

# Флоу

1. Сбор конфигурации лаяуту
2. Генерации лаяута страницы
3. Генерация блоков
4. Запуск вывода блоков
5. Вставка темплейтов
6. Запуск дочерних блоков
7. Показ вывода

В отличии от M1, в M2 ренедринг вызывается не в контролере, контролер лишь генерирует объект резльтата (реализующий [`ResultInterface`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/Controller/ResultInterface.php)), который возвращает, а затем у этого объекта вызывается метод [`renderResult()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/App/Http.php#L120).

## Пример использования объекта страницы

```php
class Index extends \Magento\Cms\Controller\Adminhtml\Block
{
    /**
     * @var \Magento\Framework\View\Result\PageFactory
     */
    protected $resultPageFactory;

    /**
     * @param \Magento\Backend\App\Action\Context $context
     * @param \Magento\Framework\Registry $coreRegistry
     * @param \Magento\Framework\View\Result\PageFactory $resultPageFactory
     */
    public function __construct(
        \Magento\Backend\App\Action\Context $context,
        \Magento\Framework\Registry $coreRegistry,
        \Magento\Framework\View\Result\PageFactory $resultPageFactory
    ) {
        $this->resultPageFactory = $resultPageFactory;
        parent::__construct($context, $coreRegistry);
    }

    public function execute()
    {
        /** @var \Magento\Backend\Model\View\Result\Page $resultPage */
        $resultPage = $this->resultPageFactory->create();
        $this->initPage($resultPage)->getConfig()->getTitle()->prepend(__('Blocks'));
        //some code
        return $resultPage;
    }
}
```
