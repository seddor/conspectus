# JQuery select elements

* ```$('#id')``` - by HTML id
* ```$('.class')``` - by CSS class
* ```$('element')``` - by HTML tag

## Retrieving element by their hierarchy

* ```div p``` - all child
* ```div > p``` - direct childs
* ```div ~ p``` - all ```p``` after any sibling ```div```
* ```div + p``` - all ```p``` after first any sibling ```div```

## Retrieving element by their attribute

* ```a[href^='#']``` - all link with ```href``` start with ```#```
* ```a[href!='#']``` - all link not equal #
* ```div[class|='main']``` - all div with 'main' and class name starting with 'main-' (like 'maint-fototer')

## filters

* ```:first```
* ```:last```
* ```:even```
* ```:odd```
* ```eq(n)``` - element with index ```n``` (may by negative)
* ```gt(n)``` - element after current exlude current
* ```lt(n)``` - element before current exlude current
Negative indexs work from the end, in JQuery filter start with 0, in CSS start with 1

## form filter

For slect all checked checkbox: ```$('input[type="checkbox"]:checked')```

## contant filter

* ```$('li:has(a):contains(Link 3)')``` - all link with ```Link 3``` text inside

# Work with attribute

Method ```attr()``` with parametrs

* (name, value). ```name``` - atribute name, ```value``` - value attribute, may be value or function
* ([attribute]). attributes is array with key => value; key - attribute name, value - value. Exemple

  ```javascript
	$('input').attr((
	  value: '',
	  title: 'New Title'
	));
  ```

all it same for properties with ```prop()``` function

# Events

* ```on(eventType[, selector][, data], handler)``` - назначить хендлер для ивента
* ```on(eventType[, selector][, data], handler)``` - назначить хендлер для ивента, который будет отловлен один раз
Для отвязки хендлов ивентов используется ```off(eventType[, selector][, handler])```
Здесь
* ```eventType``` -- тип ивента (например ```click```)
* ```selector``` -- используется для делегирования ивентов от детей, например, если нужно отлавливать нажатие на элеметы списка, то обработчик нужно назначить на их родителя

  ```javascript
  $('#my-list').on('mouseover', 'li', function(event) {
      console.log('List item: ' + ($(this).index() + 1));
  });
  ```
* ```data``` дополнительные данные для ивента, например:

  ```javascript
  $('#my-button').on('click', {
     name: 'john doe'
  }, function (event) {
      console.log('The name is: ' + event.data.name);
  });
  ```

* ```handler``` -- хендлер для ивента

Также возможно вместо этих параметров использовать eventsHash

* ```on(eventsHash[, selector][, data])```
* ```one(eventsHash[, selector][, data])```
* ```off(eventsHash, [, selector])```

Пример:

  ```javascript
  //eventType
  $('button')
    .on('click', function(event) {
       console.log('Button clicked!');
    })
    .on('mouseenter mouseleave', myFunctionHandler);

    //eventsHash
    $('button').on({
   click: function(event) {
      console.log('Button clicked!');
   },
   mouseenter: myFunctionHandler,
   mouseleave: myFunctionHandler
  });
  ```

Обычно вложеные ивенты обрабатываются от самого вложеного и верх по иерархии - "всплытие". В современых браузерах также работает и обратный порядок("Погружение").
Чтобы остановить обработку всплытия можно использовать следующие методы

* ```event.stopPropagation()``` - обрабатывает ивент на теущем элементе и предотвращает дальшейщую обработку этого ивента
* ```event.stopImmediatePropagation()``` предотвращает дальнейшую обработку ивента, а также останавливает обработку текущего элемента

Помимо отлавливание ивентов можно их тригерить

* ```trigger(eventType[, data])``` - тригерит ивент, тем самым запуская назначеные на этот ивент хендлы
* ```triggerHandler(eventType[, data])``` - тригер ивента на одном хендлере(первом попавшимся) без вызова дальнейших обработчиков

>Примечание: принято использовать для привязки событий именно `on('<event>')`, а не конкретные элиасы ивентов типа `click` и т.п.


# AJAX

* ```load(url, [data], [callback], [dateType])``` - запрос к серверу без перезагрузки страницы, если задана `data` то шлётся POST, иначе GET. В ответ приходит коллекция.
Можно даже делать так: `.load('data.html p:last-of-type', loadCb)`. Вставится только последний из абзацев.
* ```get(), post``` - для отправки пост и гет запросов соответственно
* ```ajax(url[, options]) или ajax([options])``` - возвращет jqXHR(расширенный объект XHR, с методами добавлеными джиквери) объект. [Весь список опций](http://api.jquery.com/jQuery.ajax/) подробнее ниже

## ajax()

* ```ajaxSetup(options)``` - позволяет устанавливать дефотлные опицции на все аяксовые запросы джиквери. Пример:

  ```javascript
  $.ajaxSetup({
    type: 'POST',
    timeout: 5000, //time in milliseconds
    dataType: 'html'
  });
  ```

### События ajax

 * локальные `beforeSend`, `done`, `fail`, `alway` - обрабатываются в конкретных вызовах `$.ajax` в их лямбде
 * глобальные - для всех аякс-запросов: `ajaxStart`, `ajaxSend`, `ajaxSucces`, `ajaxError`, `ajaxStop`, `ajaxComplete`. - эти события можно отлавливать в `document`, например ` $(document).on('ajaxStart)`

### Дополнительные методы

 * `$.ajaxPrefilter([dataTypes,] callback)` - обработка аяксовых запросов перед отправкой, можно прервать запрос, перенаправить или изменить `dataType`
 * `$.ajaxTransport([dataType,] callback)` - позволяет изменять внутренее устройство запросов, должен возращать объект, с методмаи `send` и `abort` [доролнительно](https://jquery-docs.ru/jQuery.ajaxTransport/)
 
