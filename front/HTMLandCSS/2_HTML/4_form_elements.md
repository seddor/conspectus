# Элементы форм

* input — позволяет создавать различные части интерфейса и обеспечить взаимдействие с пользователем.
    * type — тип элемента, по умолчанию text.
    * value — значение
    * name — имя, будет использовано как ключ при отправке на сервер.
    * placeholder — плейсхолдер, будет отображаться пока поле будет пустое.
    * disabled — блокирует измения поля.
[Подробнее](https://webref.ru/html/input)
```HTML
<input type="text" value="текстовое поле">
```
Можно так-же создавать списки, если указать несколько инпутов с одним именем.
* select — создаёт список, внутри содержит дочерние тэги `option`
    * multiple — множественный выбор
    * size — количество отображемых строк
```HTML
<select name="choise">
    <option selected>Make a choise</option>
    <option value="foo">foo</option>
    <option value="foo">bar</option>
    <option value="foo">baz</option>
</select>
```
* textarea — многострочная форма ввода текста
    * rows — количество строк
    * cols — количество символов в одной строке
```HTML
<textarea name="text">
Text
</textarea>
```
* button — кнопка
    * type — тип кнопки
        * reset — сбрасывает текстовые поля формы.
        * button — просто кнопка, ничего не делает используется для дальнейшего использования через JS.
        * submit — отправляет форму, испоозуется по умолчанию
```HTML
<button>This is button</button>
```
* label — устанавливает связь между элементом формы и другим элементом(обычно текстовым). 
Связь осуществляется посредством id элемента формы.
```HTML
<label for="text">Text:</label><br/>
<textarea id="text"></textarea>
```
