# Архитектура

M2 так-же как M1 построена по модульному принципу, однако модули M2 ещё более независимы.

# Области (Areas)

Область (area) — это скоуп конфигурации позволяющий загрузить только нужные конфигурационные файлы, опции требуемые нужному запросу. В M2 они схожи с такими в M1. 

M2 имеет такие области(`Magento\Framework\App\Area`):

* **Admin** (`adminhtml`) — всё связанное с админкой
* **Storefron** (`frontend`) — пользовательская часть
* **Basic** (`base`) — используеться для fallback если что-то не было найдено в `adminhtml` или `frontend`
* **Cron** (`crontab`) — для `cron.php` `\Magento\Framework\App\Cron` всегда загружаеться в этой области 
Так же имеються area для API
* **Web API REST** (`webapi_rest`)
* **Web API SOAP** (`webapi_soap`)
* **Graphql** (`graphql`)

# Основные концепции

1. Модульность
2. Темы — набор файлов отвечающих за внешний вид сайта, темы содержат общие статичные файлы, фронтовые файлы.
3. Layout — основной концепт системы рендеринга мадженты, это layout-файлы ­— набор XML файлов содержащий определения какие блоки будут распологаться на странице и каким образом. Структура блоков представляет из себя иерархию. Блок это PHP класс, обычно соединенный с темплейтом (но не всегда). 
4. Мердж конфигов — маджента сливает вместе все модульные конфиги одного типа.
5. "Магия" инциализации объектов — или DI (внедернение зависимостей). M2 активно использует DI для инциализации при работе с объектами, большинство зависимостей внедряются через конструкторы.
6. Именнование для контроллеров и лаяутов — ещё одна важная фича M2, для обработки роута нужно создать соответствующий контроллер (и связать его в routes.xml). Так-же чтобы добавить новый элемент на страницу нужно создать специальный layout-файл, имя которого отсылает к роуту.
7. События и плагины — две основные концепции кастомизации в M2. События возникают в коде и разработчик может их отлавливать и обрабатывать. Каждое событие имет параметры, которые можно использовать или модифицировать. Плагины позволяют добавлять специфичное поведение для не финальныхльных паблик методов каждого класса.

# Компоненты

* Configuration (app/etc) — основные конфигурации всего инстанса M2. Основные конфигурации определяют какие модули будут доступны, подключение к БД и другое. Модульные конфигурации определяют только поведение этого модуля.
* Freamwork (vendor/magento/fremwork) — содержит низкоуровневый код, не связаный с бизнес логикой. Опеределяет как система работает в целом: взаимодействие с БД, обработка URL, логирование и т.п.
* Modules (vendor/magento/module-*) — содержат бизнес логику, реализацию фич.
* Command-line Tool (bin/magento) — содержит `bin/magento` скрипт для управления инстансом M2, позволяет запускать инсталеры, отключать модули, чистить кэш и т.д.
* Themes (app/design) — темы, обычно набор статичных файлов, часть файлов тем может распологатся в модулях (например функционал JS) или в директории _app/design_.
* Dev tools (dev) — содержит скрипты и инструменты помогающие разрбаотчику, напрмер содержит фреймворк для тестирования.
* Generated (generated) — содержит класса сгенерированные M2 через DI или плагины. Очень полезны для ускорения работы системы.

# Типы файлов

* конфигурации (xml, некоторые PHP-файлы)
* PHP-классы (php)
* Лаяуты (xml)
* Шаблоны (phtml)
* JavaScript модули (js)
* JavaScript шаблоны (html)
* Статика (CSS, изображения и т.д.)

# Конфигурации

Разделяется на 3 группы;

* Глобальная (основаная) конфигурация в директории _app/etc_
* Конфигурации модулей
* Конфигарации тем

# PHP классы

Оснновные типы:

* model/resource model/collection — классы для взаимодействия с БД, определяющие повдедение сущности. Сейчас M2 стремится избегать использование этих классов в пользу интерфейсов API, однако они остаются важной частью архитектруры.
* API interface — определяют API модуля и какие операции в нём доступны. Включают CRUD операции для сущностей модуля а также другие модуле-специфичные операции, (например авторизация пользователя). M2 имеет специальные механизм для связи определённых классов с имплементацией интерфейсов API.
* Controllers — контроллеры из MVC архитектуры, предназначены для обработки определённых роутов.
* Blocks — блоки предназначены для представления части страницы, обычно связаны с phtml-шаблонами.
* Observers — обработчики событий.
* Plugins — классы для кастомизации поведения других классов, могут оборачивать любой не финальный паблик метод любого не финального класса.
* Helpers — вспомогательные классы для инкапсуляции некоторых полезных функций.
* Schema/Data patсhes — скрипты для обновления схемы или данных БД.
* UiComponents — UiComponents ещё одно нововедение M2 позвляет создавать компоненты — независимые части страницы с собственой бекенд частью.

M2 содержит так-же и другие типы классов, здесь были перечислены только основные.
