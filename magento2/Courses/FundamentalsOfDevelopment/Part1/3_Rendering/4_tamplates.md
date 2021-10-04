# Темплейты

Темплейты есть двух видов:
* *phtml* — представляют из себя phtml файлы, содержащие HTML-верстку со вставками PHP-кода.
* *html* — представляют из себя html файлы, используются для JS (knockout).

## Расположение

Темлейты находится в модулях, в дирктории _view/`<area>`_:

* _templates_ — phtml-темлейты
* _web/templates_ — html-темплейты

Так-же методы могут располагаться в темах, в поддериектории с именем модуля, при этом этот темплейт обладает большим приоритетом и используется для переопределения темлейта из модуля.

### phtml

В phtml-темплейтах можно вызывать публичные методы блока через `$block` или `$this`. Предпочтительнее использовать `$block`, `$this` считается устаревшим.

#### Magic-метод

Во время рендеринга блока темлейт включается в классе [Magento\Framework\View\TemplateEngine\Php](https://github.com/magento/magento2/blob/2.3/lib/internal/Magento/Framework/View/TemplateEngine/Php.php), в нём содержится magic-метод `__call`, который переадресует вызовы из темлейта :
```php
public function __call($method, $args)
{
    return call_user_func_array([$this->_currentBlock, $method], $args);
}
```
