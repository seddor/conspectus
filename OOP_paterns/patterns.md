# Коспект по книге паттерны проектированния

Основаная задача паттерна — предложить решение определённой задачи в конкретном контексте.

#### Содержание
- [Структурные](#structural)
  * [Composite (компоновщик)](#composite)
  * [Decorator (Декоратор)](#decorator)
  * [Adapter (Адаптер)](#adapter)
- [Поведенческие](#behavioral)
  * [Strategy (стратегия)](#strategy)   
- [Порождающие](#creational)
  * [Abstruct Factory(Абстрактная фабрика)](#abstruct-factory)
  * [Builder (Строитель)](#builder)
  * [Factory Method (Фабричный метод)](#factory-method)
  * [Prototype (Прототип)](#prototype)
  * [Singleton (Одиночка)](#singleton)

## <a name="structural"></a> Структурные

### <a name="composite"></a> Composite (компоновщик)

[В каталоге](https://refactoring.guru/ru/design-patterns/composite)
[Пример на PHP](http://designpatternsphp.readthedocs.io/ru/latest/Structural/Composite/README.html)

Позволяет сгруппировать объекты в древовидную структуру, а затем рбаотать как с единым объектом.

Паттер имеет смысл только если структура объектов программы может быть представлена в виде дерева, содержащего примитивы и контейнеры содержащие примитивы.

Основная часть паттерна это абстрактный класс(или интерфейс, в зависимости от сложности структуры) который представляет одновремено примитивы и контейнеры, он содержит базовый методы которые могуть вызваны у примитивов и контейнеров.

Например, имеем интерфейс `Graphic`

```php
<?
interface Graphic
{
  public function draw();
}

/**
* Класс примитива
*/
class Line implements Graphic
{
  public function draw()
  {
    //...
  }
}

/**
* Класс контейнера
*/
class Line implements Graphic
{
  private $elements;

  public function draw()
  {
    foreach ($this->elements as $element) {
        $element->draw();
    }
  }

  public function add(Graphic $element)
  {
    $this->elements[] = $element;
  }
}
```

В итоге мы получаем структуру в которой есть составной объект который может содержать несколько детей, которые в свою очередь тоже могут быть такими же составными объектами, при этом благодаря интерфесу у каждого объекта есть метод `draw()` и когда мы вызовем `draw()` у основного объекта у каждого элемента иерахии будет вызвана своя реализация метода

### <a name="decorator"></a> Decorator (Декоратор)

[В каталоге](https://refactoring.guru/ru/design-patterns/strategy)
[Пример на PHP](http://designpatternsphp.readthedocs.io/ru/latest/Structural/Decorator/README.html)

Предназначен для динамического добавления оюъекту новых функций, является гибкой альтрнативой наследования. По сути декоратор является обёрткой над объектом.

```php
<?

interface ComponentInterface
{
  public function execute($data);
}

/**
* Класс конкретного объекта, который будет оборачиватся в декоратор
*/
class ConcreteComponent
{
  public function execute($data)
  {
    //...
  }
}

/**
* Базовый декоратор
*/
abstract class AbstractDecorator
{
  protected $component;

  public function __constructor(ComponentInterface $component)
  {
    $this->component = $component;
  }
}

/**
* Декоратор
*/
class Decorator extends AbstractDecorator
{
  /**
  * Обертка для метода
  */
  public function execute($data)
  {
    $this->component->execute($data);
    //здесь можно изменять поведение обёрнутого объекта и всё такое
  }

  /**
  * Новый метод
  */
  public function foo()
  {
    //...
  }
}
```

Таким образом декоратор позволяет расширить какой-либо класс не прибеграя к наследованию. Преимуществом такого подхода является то, что объякт расширяется динамически и может получать нужные свойства во время выполнения.

Недостатки такого подхода
  - Трудность работы с многократно обёртнутыми объектами.
  - Может привести к обилию мелких классов декораторов.

### <a name="adapter"></a> Adapter (адаптер)

[В каталоге](https://refactoring.guru/ru/design-patterns/adapter)
[Пример на PHP](http://designpatternsphp.readthedocs.io/ru/latest/Structural/Adapter/README.html)

Преобразует интерфейсм одного класса в интерфейс другого, тем самым позволяет объектам с несовместимыми интерфейсами работать вместе.

Имеет следующюю структуру:

```               
               ____________________
(C) Client---> |(I)_ClientInterface|
               |    + method(data) |
               |___________________|      
                          ^
                __________|_______        ______________________________
                |__(C) Adapter____|       |________(C) Service_________|
                |- adptee: Service| ----> |...                         |
                |+ method(data)   |       |+ serviceMethod(specialData)|
                |_________________|       |____________________________|
```
Здесь:

- Client — класс содержащий текущую бизнес логкику
- ClientInterface — описывает протокол, через которые клиент работает с другими классами
- Adapter — класс, работающий одновременно с килентом и сервисом, Он результирует клиентские интерфейсы и содержит сслыку на объект сервиса. Получает вызовы от клиента(через методы интерфейса) и переводит их сервису, преобразую в требуемый формат и обратно.                  

```php
<?
/**
* Клиентский интерфейс
*/
interface BookInterface
{
    public function turnPage() : void;
    public function open() : void;
    public function getPage(): int;
}
/**
* Клиент
*/
class Book implements BookInterface
{
  private $page;
  public function open() : void
  {
      $this->page = 1;
  }
  public function turnPage() : void
  {
      $this->page++;
  }
  public function getPage(): int
  {
      return $this->page;
  }
}
/**
* Класс сервиса
*/
class EBook
{
    private $page = 1;
    private $totalPages = 100;
    public function pressNext() : void
    {
        $this->page++;
    }
    public function unlock() : void
    {
    }
    public function getPage(): array
    {
        return [$this->page, $this->totalPages];
    }
}
/**
* Класс адаптера
*/
class EBookAdapter implements BookInterface
{
    protected $eBook;
    public function __construct(EBookInterface $eBook)
    {
        $this->eBook = $eBook;
    }
    public function open() : void
    {
        $this->eBook->unlock();
    }
    public function turnPage() : void
    {
        $this->eBook->pressNext();
    }
    public function getPage(): int
    {
        return $this->eBook->getPage()[0];
    }
}
```

## <a name="behavioral"></a> Поведенческие

### <a name="strategy"></a> Strategy (стратегия)

[В каталоге](https://refactoring.guru/ru/design-patterns/strategy)
[Пример на PHP](http://designpatternsphp.readthedocs.io/ru/latest/Behavioral/Strategy/README.html)

Предназначен для определение семейства схожих объектов, инскапсулирует каждый из них и делает взаимпозаменяемыми. Например, существет множество алгоритмов для разбиения текста, или множнство вариантов сравнения объктов, чтобы не реализовывать сразу все алгоритмы в одном классе можно воспользоватся стратегией. Данный паттерн позволяет уйти от наследования к делигированнию.

Структура стратегии состоит в следующем: есть класс контекста, есть интерфейс стратегии и множество классов реализующих этот интерфейс.

```php
<?

interface StrategyInterface
{
  public function execute($data);
}

/**
* Класс контекста
*/
class Context
{
  private $_stategy;

  /**
  * Это метод будет вызватся в клиенте
  */
  public function doSomething($data)
  {
    //метод должен содержать вызов объекта-стратегии
    $this->strategy->execute($data)
  }

  public function setStrategy(StrategyInterface $strategy)
  {
    $this->strategy = $strategy;
  }
}

/**
* Класс стратегии 1
*/
class StategyFoo implements StrategyInterface
{
  public function execute($data)
  {
    //...
  }
}

/**
* Класс стратегии 2
*/
class StategyBar implements StrategyInterface
{
  public function execute($data)
  {
    //...
  }
}
```

Работает всё так: `Context` хранит ссылку на объект-стратегию реализующим интерфес `StrategyInterface`, объект-стратегия реализует конкретную вариацию алгоритма. Во время выполнения `Context`  вызывается клиентом и делегирует эти вызовы на конкретные объекты-стратегии, объекты-стратегии создаются клиентом и передаются контексту, сам контекст не имеет представления о текущей реализации стратегии, всё общение со стратегией у него только через интерфейс.

## <a name="creational"></a> Порождающие

### <a name="abstruct-factory"></a> Abstruct Factory(Абстрактная фабрика)

[В каталоге](https://refactoring.guru/ru/design-patterns/abstract-factory)

Абстрактная фабрика позволяет создавать симейства связаных объектов, не привязываясь к конкретным классам создаваемых объектов.

Структура

```php
<?

abstract class AbstractFactory
{
  abstract public function create();
}

/**
* Класс конкретной фабрики
*/
class fooFactory extends AbstractFactory
{
  public function create()
  {
    return new FooObject();
  }
}

/**
* Класс конкретной фабрики
*/
class barFactory extends AbstractFactory
{
  public function create()
  {
    return new BarObject()
  }
}

/**
* Базовый класс объектов
*/
abstract class Object
{
  abstract public function doSomething();
}

/**
* Рнализация объекта, соответствено так же и BarObject
*/
class FooObject extends Object
{
  public function doSomething()
  {
    //...
  }
}
```

Суть заключается в том, что клиент в зависимости от контекстаа создаёт нужную ему фабрику, которая будет возмващать нужные объекты.

Преимущества:

  - Избавляет код от привязки к конкретным классам реализациям
  - Код для производства объектов выносится в общее место
  - Упрощает добавление новых классов реализаций

Недостатки:
  - Усложняет код программы за счёт дополнительныъ классов
  - Для каждой реализации нужно реализовать все типы объектов

### <a name="builder"></a> Builder (Строитель)

[В каталоге](https://refactoring.guru/ru/design-patterns/builder)
[Пример на PHP](http://designpatternsphp.readthedocs.io/ru/latest/Creational/Builder/README.html)

Позволяет делигировать конструкирование сложных объектов на класс строитель, который позволяет пошагово сконструировать и сконфигурировать какой-либо объект.

```php
<?
/**
* Класс Director управляет строителяит, именно в нём происходит конфигурация создаваемого объекта и
* имнно он знает в каком порядке стоит взаимодействовать со строителями
* сам класс работает со строителем только чрез интрфейс и не может ничего знать о конкретной реализации строителя
*/
class Director
{
  public function build(BuilderInterface $builder): Vehicle
  {
      $builder->createVehicle();
      $builder->addDoors();
      $builder->addEngine();
      $builder->addWheel();

      return $builder->getVehicle();
  }
}
/**
* Интерфейс строителя
*/
interface BuilderInterface
{
    public function createVehicle();
    public function addWheel();
    public function addEngine();
    public function addDoors();
    public function getVehicle(): Vehicle;
}
/**
* Класс создаваемого объекта
*/
class Vehicle
{
  //...
}
/**
* Класс конкретного строителя
*/
class TrucBuilder implements BuilderInterface
{
  private $car;
  public function addDoors()
  {
      //...
  }
  public function addEngine()
  {
      //...
  }
  public function addWheel()
  {
      //...
  }
  public function createVehicle()
  {
      //...
  }
  public function getVehicle(): Vehicle
  {
      return $this->car;
  }
}
```

Суть работы с патерном, в том, что клиент вызывает метод директора build передавая ему конкретного строителя, директор в нужном порядке вызывает методы строителя для конфигурации, затем вызывает метод создания объекта у строителя и возвращает результат клиенту. Таким образом, код для создания и конфигурирования общих объектов отделается от основной бизнес логики, также благодаря строителю структурируется процесс конфигурации объектов.

### <a name="factory-method"></a> Factory Method (Фабричный метод)

[В каталоге](https://refactoring.guru/ru/design-patterns/factory-method)
[Пример на PHP](http://designpatternsphp.readthedocs.io/ru/latest/Creational/FactoryMethod/README.html)

Паттерн определяет интерфейс для создания объектов, но так же позволяет подклассам изменять тип создаваемого объекта.

```php
<?
/**
* Класс содержащий абстрактный фабричный метод,
* сам класс не обязательно является только фабирикой, он мжоет так-же содержать и другие методы
*/
class Director FactoryMethod
{
  /**
  * Абстрактный фабричный метод который будет переопределён в конкретных фабриках
  */
  abstract protected function createVehicle(string $type): VehicleInterface;

  /**
  * Общий метод создания объекта, который будет вызыватся в клиентском коде
  */
  public function create(string $type): VehicleInterface
  {
      $obj = $this->createVehicle($type);
      $obj->setColor('black');

      return $obj;
  }
}
/**
* Интерфейс конкретной фабрики
*/
class ItalianFactory extends FactoryMethod
{
    protected function createVehicle(string $type): VehicleInterface
    {
        switch ($type) {
          case 'foo':
            return new Foo();
          case 'bar':
            return new Bar();
          //...    
          default:
            throw new \InvalidArgumentException("$type is not a valid vehicle");
        }
    }
}
```

В клиенском коде создаётся экземпляр конкретной фабрики, через которую создаются объекты. Преимущетва и недостатки схожи с абстрактной фабрикой. Однако в отличии от абстрактной фабирики, фабричный метод может содержать в себе собственую логику, делегирую создани объектов на фабрики, которые его наследуют.


### <a name="prototype"></a> Prototype (Прототип)

[В каталоге](https://refactoring.guru/ru/design-patterns/prototype)
[Пример на PHP](http://designpatternsphp.readthedocs.io/ru/latest/Creational/Prototype/README.html)

Паттерн создаёт объект на основании какого-либо прототипа.
Суть паттерна в том, что он делегирует задачу клонирования объектов на сами объекты, которые нужно клонировать.

В PHP уже реализован встроенный механизм клонирования объектов, [почитать можно здесь](http://php.net/manual/ru/language.oop5.cloning.php)

Для этого в PHP использует `clone`

```
$foo = new Foo();
$fooClone = clone $foo;
```

Если у клонируемого объекта переопределён метод `__clone()` то при клонировании, после создания объекта он будет вызыван.

### <a name="singleton"></a> Singleton (Одиночка)

[В каталоге](https://refactoring.guru/ru/design-patterns/singleton)
[Пример на PHP](http://designpatternsphp.readthedocs.io/ru/latest/Creational/Singleton/README.html)

Паттерн предназначен для случаев, когда необходимо иметь только один экземпляр объекта в приложении. Данный паттерн гарантирует это и предоставляет для глобальгую точку доступа до экземпляра объекта, запрещает создавать новые экземпляры для объекта.

```php
<?
final class Singleton
{
    private static $instance;
    /**
     * Используется в клиентском коде для получения инстанса,
     */
    public static function getInstance(): Singleton
    {
        if (null === static::$instance) {
            static::$instance = new static();
        }
        return static::$instance;
    }
    /**
     * Необходимо запретить любой способ прямого или косвеного создания объекта
     * объект должен создаватся только через getInstance
     */
    private function __construct()
    {
    }
    private function __clone()
    {
    }
    private function __wakeup()
    {
    }
}
```

Достоинства
  - Гарантирует, что объект существует в едином экземпляре
  - Предоставляет к объекту глобальную точку доступа

Недостатки
  - Нарушает прицип единсвенной ответственности классов
  - Тредует постояного моканья при юнит тестировании
