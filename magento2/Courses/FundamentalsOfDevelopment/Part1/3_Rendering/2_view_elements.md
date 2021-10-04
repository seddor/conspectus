# View-элементы

Есть 3 типа view-элементов:
* UiComponents — концептуально это автономные, переиспользуемые элементы, размещаемые на странице. В M2 они используются для гридов, форм, мини-корзины и т.п.. Они ренедарятся с помощью JS, бекенд используется только для получения данных.
* Container — контейнер это элемент, которые рендерит все входящие в него элементы, при этом он не имеет блока (класса) который к нему относится. В контейнере можно настроить некоторые атрибуты (wrapping tag, css классы). Пример задавания контейеров в [`empty.xml`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Theme/view/base/page_layout/empty.xml):
```xml
<layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_layout.xsd">
    <container name="root">
        <container name="after.body.start" as="after.body.start" before="-" label="Page Top"/>
        <container name="page.wrapper" as="page_wrapper" htmlTag="div" htmlClass="page-wrapper">
            <container name="global.notices" as="global_notices" before="-"/>
            <container name="main.content" htmlTag="main" htmlId="maincontent" htmlClass="page-main">
                <container name="columns.top" label="Before Main Columns"/>
                <container name="columns" htmlTag="div" htmlClass="columns">
                    <container name="main" label="Main Content Container" htmlTag="div" htmlClass="column main"/>
                </container>
            </container>
            <container name="page.bottom.container" as="page_bottom_container" label="Before Page Footer Container" after="main.content" htmlTag="div" htmlClass="page-bottom"/>
            <container name="before.body.end" as="before_body_end" after="-" label="Page Bottom"/>
        </container>
    </container>
</layout>
```

Контейнеры можно расширять в других лаяутах, напимер в [`1column.xml`](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Theme/view/frontend/page_layout/1column.xml) расширяет `empty.xml` и добавляет новые контейнеры в существующий `page.wrapper`:
```xml
<layout xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:View/Layout/etc/page_layout.xsd">
    <update handle="empty"/>
    <referenceContainer name="page.wrapper">
        <container name="header.container" as="header_container" label="Page Header Container" htmlTag="header" htmlClass="page-header" before="main.content"/>
        <container name="page.top" as="page_top" label="After Page Header" after="header.container"/>
        <container name="footer-container" as="footer" before="before.body.end" label="Page Footer Container" htmlTag="footer" htmlClass="page-footer"/>
    </referenceContainer>
</layout>
```

* Blocks — копноненты содержаищие контент, большинство из них имеют свои темплейты. 

Каждая страница в M2 представляет собой иерархию контейнеров, которые состоят из блоков, которые могут содержать в себе любое количество дочерних блоков или контейнеров. Иерархия определяется в лаяуте, при этом лаяут не определяет местоположение блока на странице, он определяет лишь его положение в иерархии.

## Роль блоков

* изменеие внешнего вида сайта
* добавление чего-то на страницу
* измения стиля определённых элементов на странице
* изменение данных на странице

# Рендеринг корневого темлейта

Происходит в методе [`renderPage()`](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/View/Result/Page.php#L317):
```php
protected function renderPage()
{
    $fileName = $this->viewFileSystem->getTemplateFileName($this->template);
    if (!$fileName) {
        throw new \InvalidArgumentException('Template "' . $this->template . '" is not found');
    }
    ob_start();
    try {
        extract($this->viewVars, EXTR_SKIP);
        include $fileName;
    } catch (\Exception $exception) {
        ob_end_clean();
        throw $exception;
    }
    $output = ob_get_clean();
    return $output;
}
```
