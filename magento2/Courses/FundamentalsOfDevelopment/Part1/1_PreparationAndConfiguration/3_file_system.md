# Файловая система

## Коневые директории

* _app_ — содержит код ядра (если ставить M2 клонированием из git-репозитория), кастомные модели, темы и глобалную конфигурацию.
* _bin_ — содержит shell-скрипт magento позволяющий выполнять рзаличные системые команды M2, например миграции, отключение модулей, очистка кеша и т.д
* _dev_ — скрипты и инструменты для разработчиков
* _lib_ — внешние библиотеки, не устанавливаемые через composer, если ставить M2 клонированием из git-репозитория, то содержит так-же файлы фреймворка
* _vendor_ — директория composer'а содержит все зависимости проета, а так-же файлы M2 при установке через composer
* _pub_ — точка вхождения приложения через web, содержит директорию _static_ куда сохраняются сатичные файлы в проде, обычно эти файлы содержатся в модулях и темах и перемещаются в _pub/static_ по команде, так-же есть `static.php` который находит и возвращает статичные файлы модулей и тем. В прод режиме будет содержать копии файлов, в девелоп-режиме симлинки на файлы из темы/модуля.
* _setup_ — содержит файлы для установки
* _var_ — содержит логи, кеш и т.д.

### App

* _app/etc_ — содержит глобалную конфигурацию
* _app/code_ — кастомные модули, и модули ядра (если M2 клонирована из git-репозитория)
* _app/design_ — темы
* _app/i18n_ — файлы локализации
* _app/bootstrap.php_, _app/autoload.php_, _app/functions.php_ — особые PHP-файлы участвующие в начале поцесса исполнения.

## View-файлы

В отличии от M1, в M2 лаяуты и шаблоны хранятся в _view_ директории модуля, что делает модуль более независимым и целостным, т.к. все файлы модуля хранятся в одной директории.

_View_ директория содержит в себе поддиректории с названием области(area), такими как: _frontend_, _adminhtml_, _base_. Внутри них содеражтся директори: 
* _templates_ — содержит phtml-шаблоны.
* _layout_ — содержит layout-файлы.
* _page_layout_ — содержит page layout файлы.
* _web_ — содержит изображения, стили, JS.

## Темы

В _app/design_ (или _vender_ при добавлении через composer) находятся темы, темы содержат в себе статичные файлы, шаблоны, лаяуты.