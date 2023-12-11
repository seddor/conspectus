# Веб-разработка

В go есть готовые библиотеки для обработки веб-запросов.

## Реализация веб-приложения

С помощью стандартной библиотеки [`net/http`](https://pkg.go.dev/net/http) можно написать веб-приложение:

```go
package main

import (
    "net/http"
)

func main() {
    // обозначаем, что на запрос по пути "/" возвращается строка "Hello World"
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        // тело ответа — это массив байт
        w.Write([]byte("Hello world!"))
    })

    // запускаем веб-приложение для обработки запросов
    http.ListenAndServe(":80", nil)
}
```

Теперь если перейти на http://localhost/ то там будет страница с надписью "Hello world!".

Функция `http.HandleFunc` настраивает обработчик по заданному пути, в нашем случае это `/`.

Функция `http.ListenAndServe` запускает веб-приложение на **80** порту и слушает входящие запросы. Все входящие запросы будут отработаны согласно логики, описанной в обработчиках `http.HandleFunc`.

Все бизнес логика приложения находится внутри **обработчиков**.

## Сопоставление пути и обработчика

Сервер получает запрос, смотри его пути и по нему определяет обработчика. Это процесс называет **маршрутизацией** или **роутингом**.

Чтобы использовать роутинг разилчных запросов не уровне Go нужно описать обработчик под каждый пути запроса:

```go
package main

import (
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Welcome to Hexlet"))
    })

    http.HandleFunc("/about", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hexlet is the leading educational platform for Software Engineers"))
    })

    http.ListenAndServe(":80", nil)
}
```

## Параметры запросов

При получении запрос веб-приложени на `net/http` считывает всю информацию и заполняет объект `r *http.Request`:

```go
package main

import (
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Welcome to Hexlet"))
    })

    http.HandleFunc("/courses", func(w http.ResponseWriter, r *http.Request) {
        // считываем параметр page из запроса
        page := r.URL.Query().Get("page")

        // рассчитываем, какую страницу нужно вернуть
        ...

        // возвращаем курсы страницы
        w.Write(pageCourses)
    })

    http.ListenAndServe(":80", nil)
}
```

## Логирование

## Стандартный пакет `log`

В Go есть стандартный пакет для легирования `log`, с его помощью можно:

* Установить место для записи логов;
* Писать логи построчно;
* Заканчивать выполнение всей программы в случае фатальных ошибок.

```go
// Возвращаем объект глобального логгера, который
// по умолчанию пишет логи в стандартный вывод
// операционной системы (os.Stdout)
l := log.Default()
// Меняем вывод логов в стандартный вывод ошибок
// операционной системы
l.SetOutput(os.Stderr)
// Логируем строку "something went wrong" в stderr
l.Println("something went wrong")
// Логируем строку "something went wrong" в stderr
// и паникуем с тем же сообщением
l.Panicln("something went wrong with panic")
// Логируем строку "something went wrong" в stderr
// и завершаем выполнение программы с тем же сообщением
l.Fatalln("something went wrong with fatal error")
```

Пример использования легирования:

```go
package main

import (
    "log"
    "net/http"
)

func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        _, err := w.Write([]byte("Welcome to Hexlet"))
        if err != nil {
            log.Printf("welcome to hexlet error: %s\n", err.Error())
        }
    })

    port := "80"
    log.Println("Starting a web-server on port " + port)
    log.Fatal(http.ListenAndServe(":"+port, nil))
}
```

Теперь по первом запуске будет вывод:

```bash
$ go run main.go
2022/04/13 11:02:59 Starting a web-server on port 80
```

А при попытке поднять второй сервис отобразится ошибка:

```bash
$ go run main.go
2022/04/13 11:03:51 Starting a web-server on port 80
2022/04/13 11:03:51 listen tcp :80: bind: address already in use
exit status 1
```

`log` подхоит для простого логирования, как в примере выше, но в реальных приложениях обычно его не используются из-за недостатков:

* Нет четкого раздления на уровни логирования (debug, info, error и т.п.);
* Пакет работает только со строками. Если нужно вывести другой тип данных, нужно конвертировать его в строку.

### Сторонний пакет для логирования `logrus`

[`logrus`](https://github.com/sirupsen/logrus) решает недостаки стандартного пакета `log`, в нём есть функции, чтобы записывать логи на разных уровнях. Ещё с помощью `logrus` можно вносить в сообщения дополнительную информацию любого типа данных.

```go
// Меняем вывод логов в стандартный вывод операционной системы
// Здесь можно установить любой вывод, реализующий
// интерфейс io.Writer
logrus.SetOutput(os.Stdout)
// Логируем строку "info message" на уровне INFO
logrus.Info("info message")
// Логируем строку "something went wrong" на уровне ERROR
logrus.Error("something went wrong")
// Логируем строку "something went wrong" на уровне ERROR
// вместе с ошибкой с текстом "invalid"
logrus.WithError(errors.New("invalid")).Error("something went wrong")
// Логируем строку "debug info" на уровне DEBUG
// с дополнительной информацией
logrus.WithFields(logrus.Fields{
    "count":   22,
    "time":    time.Now(),
    "message": "hello",
}).Debug("debug info")
```

Применения `logrus`:

 ```go
 package main
 
 import (
     "github.com/sirupsen/logrus"
     "net/http"
 )
 
 func main() {
     http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
         _, err := w.Write([]byte("Welcome to Hexlet"))
         if err != nil {
             // Ошибка логируется функцией WithError
             logrus.WithError(err).Error("write hello hexlet")
         }
     })
 
     port := "80"
     // Дополнительная информация передается функцией WithFields
     logrus.WithFields(logrus.Fields{
         "port": port,
     }).Info("Starting a web-server on port")
     logrus.Fatal(http.ListenAndServe(":"+port, nil))
 }
 ```

Теперь при воспроизведениии ошибки будет такой вывод:

```bash
INFO[0000] Starting a web-server on port                 port=80
FATA[0000] listen tcp :80: bind: address already in use
exit status 1
```

## Микрофреймворк Fiber

[Fiber](https://gofiber.io/) —мирофреймворк на Go для насписания веб-приложений.

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
)

func main() {
    webApp := fiber.New()
    // Обозначаем, что на GET запрос по пути /address нужно вернуть строку с адресом
    webApp.Get("/address", func(c *fiber.Ctx) error {
        return c.SendString("145 DUNDEE SOUTH SAN FRANCISCO CA 94080-1023. USA")
    })

    // Запускаем веб-приложение на порту 80
    // Оборачиваем в функцию логирования, чтобы видеть ошибки, если они возникнут
    logrus.Fatal(webApp.Listen(":80"))
}
```

Теперь при запуске веб-приложения и открытии `/address` в брузере выведится:

```
145 DUNDEE SOUTH SAN FRANCISCO CA 94080-1023. USA
```

### Запросы и ответы в Fiber

В fiber запрос и ответ содержатся в одной структуре `*fiber.Ctx`.

Представим, что нам нужно реализовать веб-приложение, у которого есть страница с профилями пользователей. Когда программа получает профиль,  передается его идентификатор с помощью GET параметра `profile_id`.

Если `profile_id` не указан, то приложение должно возвращать ответ со статусом `422` (Unprocessable Entity) и текстом ошибки в теле. Если передается корректный `profile_id`, то возвращается строка с идентификатором профиля:

```go
package main

import (
    "fmt"
    "net/http"

    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
)

const profileUnknown = "unknown"

func main() {
    webApp := fiber.New()
    webApp.Get("/profiles", func(c *fiber.Ctx) error {
        profileID := c.Query("profile_id", profileUnknown)
        if profileID == "" {
            profileID = profileUnknown
        }

        if profileID == profileUnknown {
            return c.Status(http.StatusUnprocessableEntity).SendString("profile_id is required")
        }

        return c.SendString(fmt.Sprintf("User Profile ID: %s", profileID))
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

### Роутинг в fiber

[Routing](https://docs.gofiber.io/guide/routing/)

### По пути HTTP-запроса

Правила сопоставления пути HTTP-запроса с обработчиком описываются в коде при инициализации веб-приложения:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
)

func main() {
    webApp := fiber.New()

    webApp.Get("/path1", path1Handler)
    webApp.Get("/path2", path2Handler)
    ...

    logrus.Fatal(webApp.Listen(":80"))
}
```

Когда приходит запрос от клиента веб-приложение ищет соответвующих обработчик по пути запроса.

### По методам HTTP-запроса

В Fiber роутинг по HTTP-методу запроса настраивается с помощью специальных функций:

- `webApp.Get()` — для роутинга GET-запросов
- `webApp.Post()` — для роутинга POST-запросов

Пример: есть онлайс счетчик с обрабатывающий такие запросы:

- *POST /counter*, который увеличивает счетчик на единицу
- *GET /counter*, который возвращает текущее значение счетчика и не изменяет его состояние

Реализация:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
    "net/http"
    "strconv"
)

var counter int64 = 0

func main() {
    webApp := fiber.New()

    webApp.Get("/counter", func(c *fiber.Ctx) error {
        return c.SendString(strconv.FormatInt(counter, 10))
    })

    webApp.Post("/counter", func(c *fiber.Ctx) error {
        counter++

        return c.SendStatus(http.StatusOK)
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

### Динамичный роутинг

Предположим нам нужно использовать счётчики для разных типов событий, чтобы это следать можно исползьовать динамичный рутин, для этого в путь добавляется часть, изменяемая в зависимости от требуемого счетчика: `/counter/:event`:

```go 
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
    "net/http"
    "strconv"
)

var counters = make(map[string]int64)

const requestParamKeyEvent = "event"

func main() {
    webApp := fiber.New()

    webApp.Get("/counter/:event", func(c *fiber.Ctx) error {
        event := c.Params(requestParamKeyEvent, "")
        if event == "" {
            return c.SendStatus(http.StatusUnprocessableEntity)
        }

        eventCounter, ok := counters[event]
        if !ok {
            return c.SendStatus(http.StatusNotFound)
        }

        return c.SendString(strconv.FormatInt(eventCounter, 10))
    })

    webApp.Post("/counter/:event", func(c *fiber.Ctx) error {
        event := c.Params(requestParamKeyEvent, "")
        if event == "" {
            return c.SendStatus(http.StatusUnprocessableEntity)
        }

        counters[event] += 1

        return c.SendStatus(http.StatusOK)
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

## Сериализация данных в JSON

Для сериализации и десериализации данных в JSON используется стандартная библиотека. Для этого в ней есть две функции:

* `json.Marshal()` — сериализует данные в JSON
* `json.Unmarshal()` — десериализует данные из JSON

Пример:

```go
package main

import (
    "encoding/json"
    "fmt"
    "log"
)

type User struct {
    ID       int64  `json:"id"`
    Email    string `json:"email"`
    Name     string `json:"name"`
    Age      int    `json:"age,omitempty"`
    password string
}

func main() {
    user := User{
        ID:       222,
        Email:    "john@doe.com",
        Name:     "John",
        Age:      25,
        password: "secret",
    }

    // Сериализация структуры в строку
    jsonBytes, err := json.Marshal(&user)
    if err != nil {
        // Используем Fatal только для примера,
        // нельзя использовать в реальных приложениях
        log.Fatalln("marshal ", err.Error())
    }

    fmt.Printf("JSON string: %s\n", string(jsonBytes))

    // Десериализация строки в структуру
    deserializedUser := User{}
    err = json.Unmarshal(jsonBytes, &deserializedUser)
    if err != nil {
        // Используем Fatal только для примера,
        // нельзя использовать в реальных приложениях
        log.Fatalln("unmarshal ", err.Error())
    }

    fmt.Printf("Deserialized user struct: %+v\n", deserializedUser)
}
```

Вывод программы:

```json
JSON string: {"id":222,"email":"john@doe.com","name":"John","age":25}
Deserialized user struct: {ID:222 Email:john@doe.com Name:John Age:25 password:}
```

В примере мы преобразуем объкт пользователя _User_ в JSON-структуру и обратно. В результате получается такая же структура _User_, но с пустым полем `password`, т.к. оно приватное.

Важные моменты работы с JSON в Go:

- В структуре сериализуются только публичные свойства, которые написаны с заглавной буквы. В нашем примере свойство `password` — приватное, поэтому оно не попало в JSON-строку. Такое же правило действует и на десериализацию. Если мы добавим значение `password` в JSON-строку, то это значение все равно не попадет в десериализованную структуру
- В JSON-строке можно указывать название свойства, которое будет  отличаться от названия свойства в структуре. Для этого используется тег `json:""`. В нашем примере мы указали тег для свойства `ID` в виде `json:"id"`. Так можно указывать, в каком виде должно называться свойство в JSON-строке в зависимости от соглашений в проекте
- При описании тега `json:""` можно указать опцию `omitempty`. Эта опция говорит о том, что если значение свойства является нулевым, то это поле не нужно включать в сериализованную строку

### JSON в веб-приложении

 В Fiber работа с JSON происходит через контекст запроса `c *fiber.Ctx`. Для этого есть два метода:

- `c.BodyParser()` — преобразует JSON-тело запроса в объект
- `c.JSON()` — отправляет JSON-строку в теле ответа

Реализуем веб-приложение для сохранения логов, в котором отправим JSON в теле запроса и получим JSON в теле ответа.

В приложении будет единственный метод `/logs`, который будет принимать POST-запросы на сохранение записи. В теле  запроса передается структура одной записи логов. В ответе веб-приложение возвращает структуру с идентификатором сохраненной записи:

```go
package main

import (
    "fmt"
    "github.com/gofiber/fiber/v2"
    "github.com/google/uuid"
    "github.com/sirupsen/logrus"
)

type (
    CreateLogEntryRequest struct {
        Message   string `json:"message"`
        Level     string `json:"level"`
        Timestamp int64  `json:"timestamp"`
    }

    CreateLogEntryResponse struct {
        ID string `json:"id"`
    }

    LogEntry struct {
        ID        string
        Message   string
        Level     string
        Timestamp int64
    }
)

var logs []LogEntry

func main() {
    webApp := fiber.New()

    webApp.Post("/logs", func(c *fiber.Ctx) error {
        var request CreateLogEntryRequest
        if err := c.BodyParser(&request); err != nil {
            return fmt.Errorf("body parser: %w", err)
        }

        logEntry := LogEntry{
            ID:        uuid.New().String(),
            Message:   request.Message,
            Level:     request.Level,
            Timestamp: request.Timestamp,
        }

        // Упрощенное хранение в памяти приложения
        logs = append(logs, logEntry)

        return c.JSON(CreateLogEntryResponse{
            ID: logEntry.ID,
        })
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

Теперь отправим ттестовый запрос на сохранение:

```bash
curl --location --request POST 'http://localhost/logs' \
--header 'Content-Type: application/json' \
--data-raw '{"message": "test", "level": "info", "timestamp": 1663251297}'
```

В ответ придёт идентификатор:

```bash
HTTP/1.1 200 OK

{"id":"8a53055f-6fcc-435a-b2be-35abc978a9ed"}
```

### Ссылки

1. [JSON](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON)
2. [JSON and Go](https://go.dev/blog/json)
3. [Fiber JSON Response](https://docs.gofiber.io/api/ctx#json)
4. [Fiber Body Parser](https://docs.gofiber.io/api/ctx#bodyparser)

## Хранение данных в памяти Go-приложений

Пример приложение хранящего данные в пямити:

```go
package main

import (
    "errors"
    "fmt"
    "github.com/gofiber/fiber/v2"
    "github.com/google/uuid"
    "github.com/sirupsen/logrus"
)

// Запросы и ответы
type (
    CreateOrderRequest struct {
        UserID     int64   `json:"user_id"`
        ProductIDs []int64 `json:"product_ids"`
    }

    CreateOrderResponse struct {
        ID string `json:"id"`
    }

    GetOrderResponse struct {
        ID         string  `json:"id"`
        UserID     int64   `json:"user_id"`
        ProductIDs []int64 `json:"product_ids"`
    }
)

func main() {
    orderHandler := &OrderHandler{
        storage: &OrderStorage{
            orders: make(map[string]Order),
        },
    }

    webApp := fiber.New()
    webApp.Post("/orders", orderHandler.CreateOrder)
    webApp.Get("/orders/:id", orderHandler.GetOrder)

    logrus.Fatal(webApp.Listen(":80"))
}

// Абстрактное хранилище
type OrderCreatorGetter interface {
    CreateOrder(order Order) (string, error)
    GetOrder(id string) (Order, error)
}

// Обработчик
type OrderHandler struct {
    storage OrderCreatorGetter
}

func (h *OrderHandler) CreateOrder(c *fiber.Ctx) error {
    var request CreateOrderRequest
    if err := c.BodyParser(&request); err != nil {
        return fmt.Errorf("body parser: %w", err)
    }

    order := Order{
        ID:         uuid.New().String(),
        UserID:     request.UserID,
        ProductIDs: request.ProductIDs,
    }

    id, err := h.storage.CreateOrder(order)
    if err != nil {
        return fmt.Errorf("create order: %w", err)
    }

    return c.JSON(CreateOrderResponse{
        ID: id,
    })
}

func (h *OrderHandler) GetOrder(c *fiber.Ctx) error {
    id := c.Params("id")

    order, err := h.storage.GetOrder(id)
    if err != nil {
        return fmt.Errorf("get order: %w", err)
    }

    return c.JSON(GetOrderResponse(order))
}

// Модель заказа
type Order struct {
    ID         string
    UserID     int64
    ProductIDs []int64
}

// Хранилище
type OrderStorage struct {
    orders map[string]Order
}

func (o *OrderStorage) CreateOrder(order Order) (string, error) {
    o.orders[order.ID] = order

    return order.ID, nil
}

// Ошибки
var (
    errOrderNotFound = errors.New("order not found")
)

func (o *OrderStorage) GetOrder(id string) (Order, error) {
    order, ok := o.orders[id]
    if !ok {
        return Order{}, errOrderNotFound
    }

    return order, nil
}
```

Запрос на сохранение

```bash
curl --location --request POST 'http://localhost/orders' \
--header 'Content-Type: application/json' \
--data-raw '{
    "user_id": 222,
    "product_ids": [1,2,3]
}'

```

Запрос на получение:

```bash
curl -v 'http://localhost/orders/4b49e11e-6f59-432e-9075-3aa429fd935a'

HTTP/1.1 200 OK
{"id":"4b49e11e-6f59-432e-9075-3aa429fd935a","user_id":222,"product_ids":[1,2,3]}
```

### Защита мап с мьютексом

В примере выше есть упущение. Каждый HTTP-запрос рабтает в своей горутине, что может привести к ситуации когда несколько горутин будут одновременно пытаться изменить состояние одной структуры.

Чтобы избежать проблем с этим можно использовать мьютексы. Мьютекс - это механизм синхронизации, который позволяет блокировать  доступ к критической секции кода, чтобы только одна горутина могла  читать или изменять данные в мапе в один момент времени.

Пример использования мьютекса:

```go
import "sync"

var (
    mu = sync.Mutex{}
    i = 0
)

func safeInc() {
    mu.Lock()
    i++
    mu.Unlock()
}
```

Структура мьютекс предоставляет две функции: заблокировать и разблокировать. В примере была заблокирована секция инкремента переменной `i`, что гарантирует изменение переменной только одной горутиной в один момент времени.

Приме с испольозванием мьютексов для *OrderStorage* из первого примера:

```go
type OrderStorage struct {
    mu     sync.Mutex
    orders map[string]Order
}

func (o *OrderStorage) CreateOrder(order Order) (string, error) {
    o.mu.Lock()
    defer o.mu.Unlock()

    o.orders[order.ID] = order

    return order.ID, nil
}

func (o *OrderStorage) GetOrder(id string) (Order, error) {
    o.mu.Lock()
    defer o.mu.Unlock()

    order, ok := o.orders[id]
    if !ok {
        return Order{}, errOrderNotFound
    }

    return order, nil
}
```

Здесь добавилась блокировка на каждый метод, который читает или пишет в мапу `orders`.

## GRUD-операции в Fiber

### CRUD-операции

Любой объект в хранилище можно представить в виде ресурса. С ресурсом можно работать посредством следующих операций:

- Создание — create
- Чтение — read
- Обновление — update
- Удаление — delete

Например, мы разрабатываем систему управления сотрудников компании.  Ресурс в данной системе — сотрудник. Путь до него мы определяем как `/employees`, а CRUD будет выглядеть следующим образом:

- C (create) — POST /employees
- R (read) — GET /employees или GET /employees/:id
- U (update) — PATCH/PUT /employees/:id
- D (delete) — DELETE /employees/:id

### GRUD-операции в Fiber

В Fiber каждый HTTP-метод представлен свей функцией:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
)

func main() {
    webApp := fiber.New()

    // Создание сотрудника
    webApp.Post("/employees", ...)

    // Получение сотрудника
    webApp.Get("/employees/:id", ...)

    // Обновление сотрудника
    webApp.Patch("employees/:id", ...)

    // Удаление сотрудника
    webApp.Delete("employees/:id", ...)

    logrus.Fatal(webApp.Listen(":80"))
}
```

Для каждой GRUD-операции описывается уникальный обработчик. Обработчик выполняет конкретную маленькую задачу и не должен содержать в себе логики, которая не относится к этой операции.

Для примера возьмём такую структуру сотрудника:

```go
type Employee struct{
    ID string
    Email string
    Role string
}

type MemoryEmployeeStorage struct{
    employees map[string]Employee
}
```

#### Создание

Для этого используется метод `POST /employees`:

```go
package main

import (
    "fmt"
    "github.com/gofiber/fiber/v2"
    "github.com/google/uuid"
    "github.com/sirupsen/logrus"
)

// Создание сотрудника
type (
    CreateEmployeeRequest struct {
        Email string `json:"email"`
        Role  string `json:"role"`
    }

    CreateEmployeeResponse struct {
        ID string `json:"id"`
    }
)

// Хранилище
type (
    Employee struct {
        ID    string
        Email string
        Role  string
    }

    EmployeeStorageInMemory struct {
        employees map[string]Employee
    }
)

func (s *EmployeeStorageInMemory) Create(empl Employee) (string, error) {
    // Генерируем ID для сотрудника
    empl.ID = uuid.New().String()

    s.employees[empl.ID] = empl

    return empl.ID, nil
}

func main() {
    webApp := fiber.New()

    storage := &EmployeeStorageInMemory{
        employees: make(map[string]Employee),
    }

    // Создание сотрудника
    webApp.Post("/employees", func(ctx *fiber.Ctx) error {
        // Парсим JSON-тело запроса в объект CreateEmployeeRequest
        var req CreateEmployeeRequest
        if err := ctx.BodyParser(&req); err != nil {
            return fmt.Errorf("body parser: %w", err)
        }

        // Сохраняем объект сотрудника в хранилище
        // Метод Create возвращает ID созданного сотрудника
        id, err := storage.Create(Employee{
            Email: req.Email,
            Role:  req.Role,
        })
        if err != nil {
            return fmt.Errorf("create in storage: %w", err)
        }

        // Возвращаем ID сотрудника JSON-строкой в теле ответа
        return ctx.JSON(CreateEmployeeResponse{ID: id})
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

#### Чтение

Метод чтения разделяется на два типа:

- Получение всех сотрудников. Для этого используется метод `GET /employees`
- Получение конкретного сотрудника. Для этого используется метод `GET /employees/:id`, где `:id` — это идентификатор сотрудника

Для это операции понадобиться создать новые объекты ответов:

```go
type (
    ListEmployeesResponse struct {
        Employees []EmployeePayload `json:"employees"`
    }

    GetEmployeeResponse struct {
        EmployeePayload
    }

    EmployeePayload struct {
        ID    string `json:"id"`
        Email string `json:"email"`
        Role  string `json:"role"`
    }
)
```

Реализация операций:

```go
    // Получение списка сотрудников
    webApp.Get("/employees", func(ctx *fiber.Ctx) error {
        // Получаем список всех сотрудников из хранилища
        employees := storage.List()

        // Формируем ответ
        resp := ListEmployeesResponse{
            Employees: make([]EmployeePayload, len(employees)),
        }
        for i, empl := range employees {
            resp.Employees[i] = EmployeePayload(empl)
        }

        // Возвращаем список сотрудников JSON-строкой в теле ответа
        return ctx.JSON(resp)
    })

    // Получение одного сотрудника
    webApp.Get("/employees/:id", func(ctx *fiber.Ctx) error {
        empl, err := storage.Get(ctx.Params("id"))
        if err != nil {
            return fiber.ErrNotFound
        }

        // Возвращаем данные сотрудника JSON-строкой в теле ответа
        return ctx.JSON(GetEmployeeResponse{EmployeePayload(empl)})
    })
```

Также небоходимо доработать хранилище данных для чтения:

```go
func (s *EmployeeStorageInMemory) List() []Employee {
    // Инициализируем массив с размером равным количеству
    // всех сотрудников в хранилище
    employees := make([]Employee, 0, len(s.employees))

    for _, empl := range s.employees {
        employees = append(employees, empl)
    }

    return employees
}

func (s *EmployeeStorageInMemory) Get(id string) (Employee, error) {
    empl, ok := s.employees[id]
    if !ok {
        // Возвращаем ошибку, если сотрудника с таким
        // идентификатором не существует
        return Employee{}, errors.New("employee not found")
    }

    return empl, nil
}
```

#### Обновление

Для обновление будет использоваться метод `PATCH /employees/:id`, для этого создаём новый объекст для запроса обновления:

```go
type (
    UpdateEmployeeRequest struct {
        Email string `json:"email"`
        Role  string `json:"role"`
    }
)
```

Реализуем обработчик:

```go
webApp.Patch("/employees/:id", func(ctx *fiber.Ctx) error {
    // Парсим JSON-тело запроса в объект UpdateEmployeeRequest
    var req UpdateEmployeeRequest
    if err := ctx.BodyParser(&req); err != nil {
        return fmt.Errorf("body parser: %w", err)
    }

    // Обновляем данные сотрудника в хранилище. Эта функция может вернуть ошибку, 
    // если сотрудника с таким идентификатором не существует.
    err = storage.Update(ctx.Params("id"), req.Email, req.Role)
    if err != nil {
        return fmt.Errorf("update: %w", err)
    }

    return nil
})
```

И добавляем методы в хранилище:

```go
func (s *EmployeeStorageInMemory) Update(id, email, role string) error {
    empl, ok := s.employees[id]
    if !ok {
        // Возвращаем ошибку, если сотрудника с таким
        // идентификатором не существует
        return errors.New("employee not found")
    }

    // Обновляем электронную почту сотрудника,
    // если новое значение было передано
    if email != "" {
        empl.Email = email
    }
    // Обновляем роль сотрудника,
    // если новое значение было передано
    if role != "" {
        empl.Role = role
    }

    s.employees[empl.ID] = empl

    return nil
}
```

#### Удаление

Для удаления будет использоваться метод `DELETE /employees/:id`, где `:id` — это идентификатор сотрудника

Реализуем обработчик:

```go
    // Удаление сотрудника
    webApp.Delete("/employees/:id", func(ctx *fiber.Ctx) error {
        storage.Delete(ctx.Params("id"))

        // Возвращаем успешный ответ без тела
        return ctx.SendStatus(fiber.StatusNoContent)
    })
```

Дорабатываем хранилище:

```go
func (s *EmployeeStorageInMemory) Delete(id string) {
    delete(s.employees, id)
}
```

## Валидация HTTP-запросов

### Ручная проверка

Пример валидации при сохранеии поста:

```go
package main

import (
    "errors"
    "fmt"
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
)

type CreatePostRequest struct {
    UserID int64  `json:"user_id"`
    Text   string `json:"text"`
}

func (req *CreatePostRequest) Validate() error {
    if req.UserID < 0 {
        return errors.New("user ID cannot be less than 0")
    }
    if req.Text == "" {
        return errors.New("text is empty")
    }
    if len(req.Text) > 140 {
        return errors.New("text is too long")
    }

    return nil
}

func main() {
    webApp := fiber.New()

    webApp.Post("/posts", func(ctx *fiber.Ctx) error {
        // Парсинг JSON-строки из тела запроса в объект.
        var req CreatePostRequest
        if err := ctx.BodyParser(&req); err != nil {
            return fmt.Errorf("body parser: %w", err)
        }

        // Проверка запроса на корректность.
        err := req.Validate()
        if err != nil {
            return ctx.Status(fiber.StatusUnprocessableEntity).SendString(err.Error())
        }

        // @TODO Сохранение поста в хранилище.

        return ctx.SendStatus(fiber.StatusOK)
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

Такой подход прост и понятен, но он плохо масштабируется: чем больше будет полей тем более грамоздким будет код, а также появится дублирование.

К тому же в этой реализации возвращаетс только одна ошибка, а не сразу все.

Чтобы это исправить можно использовать готовые библиотеки для валидации, например [go-playground/validator](https://github.com/go-playground/validator).

### Валидация с помощью validator

Эта библиотека позволяет реализовать валидацию с помощью анотаций полей структур.

Пример валидации поста:

```go
package main

import (
    "fmt"
    "github.com/go-playground/validator/v10"
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
)

type CreatePostRequest struct {
    // Описываем правила валидации в аннотациях полей структуры.
    UserID int64  `json:"user_id" validate:"required,min=0"`
    Text   string `json:"text" validate:"required,max=140"`
}

func main() {
    webApp := fiber.New()

    validate := validator.New()

    webApp.Post("/posts", func(ctx *fiber.Ctx) error {
        // Парсинг JSON-строки из тела запроса в объект.
        var req CreatePostRequest
        if err := ctx.BodyParser(&req); err != nil {
            return fmt.Errorf("body parser: %w", err)
        }

        // Проверка запроса на корректность.
        err := validate.Struct(req)
        if err != nil {
            return ctx.Status(fiber.StatusUnprocessableEntity).SendString(err.Error())
        }

        // @TODO Сохранение поста в хранилище.

        return ctx.SendStatus(fiber.StatusOK)
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

Библиотека уже содержит в себе множество готовых правил валидации. Например, можно проверять что поле является корректным емейлом:

```go
package main

import (
    "fmt"
    "github.com/go-playground/validator/v10"
)

type User struct {
    Email string `validate:"required,email"`
}

func main() {
    v := validator.New()

    // Вывод: Key: 'User.Email' Error:Field validation for 'Email' failed on the 'required' tag
    fmt.Println(v.Struct(&User{}))
    // Вывод: Key: 'User.Email' Error:Field validation for 'Email' failed on the 'email' tag
    fmt.Println(v.Struct(&User{Email: "test"}))
    // Пустой вывод, так как ошибки нет.
    fmt.Println(v.Struct(&User{Email: "test@gmail.com"}))
}
```

[Полный список проверок](https://pkg.go.dev/github.com/go-playground/validator/v10)

### Пользовательские валидаторы

Для добавления своих правил валидации используется функция `validate.RegisterValidation()`.

Например, нужно проверить что в посте отсутвуют слова из списка:

```go
package main

import (
    "fmt"
    "github.com/go-playground/validator/v10"
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
    "log"
    "strings"
)

type CreatePostRequest struct {
    // Описываем правила валидации в аннотациях полей структуры.
    UserID int64  `json:"user_id" validate:"required,min=0"`
    Text   string `json:"text" validate:"required,max=140,allowable_text"`
}

var forbiddenWords = []string{
    "umbrella",
    "shinra",
}

func main() {
    webApp := fiber.New()

    validate := validator.New()
    vErr := validate.RegisterValidation("allowable_text", func(fl validator.FieldLevel) bool {
        // Проверяем, что текст не содержит запрещенных слов.
        text := fl.Field().String()
        for _, word := range forbiddenWords {
            if strings.Contains(strings.ToLower(text), word) {
                return false
            }
        }

        return true
    })
    if vErr != nil {
        log.Fatal("register validation ", vErr)
    }

    webApp.Post("/posts", func(ctx *fiber.Ctx) error {
        // Парсинг JSON-строки из тела запроса в объект.
        var req CreatePostRequest
        if err := ctx.BodyParser(&req); err != nil {
            return fmt.Errorf("body parser: %w", err)
        }

        // Проверка запроса на корректность.
        err := validate.Struct(req)
        if err != nil {
            return ctx.Status(fiber.StatusUnprocessableEntity).SendString(err.Error())
        }

        // @TODO Сохранение поста в хранилище.

        return ctx.SendStatus(fiber.StatusOK)
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

[Fiber validation](https://docs.gofiber.io/guide/validation/)

## HTTP Middleware

Посредники или middlewares в Fiber являются функциями `func(c *fiber.Ctx) error`, которые устанавливаются перед обработчиками. Посредник может изменять запрос, добавлять заголовки, выполнять легирование, а также прервать обработку запроса.

Пример обработчика, проверяющего доступ к веб-приложению:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
)

func main() {
    webApp := fiber.New()
    webApp.Use(accessMiddleware)
    webApp.Post("/do/something", func(ctx *fiber.Ctx) error {
        ...
    })

    logrus.Fatal(webApp.Listen(":80"))
}

func accessMiddleware(c *fiber.Ctx) error {
    accessToken := c.Params("access_token")
    if !hasAccess(accessToken) {
        // Посредник прерывает цепочку обработки запроса.
        return c.SendStatus(fiber.StatusUnauthorized)
    }

    // Пользователь имеет доступ, продолжаем выполнение запроса.
    return c.Next()
}
```

Здесь создаётся посредник `accessMiddleware`, проверяющий Наличе токена доступа в параметрах запроса. Если токен не прошёл проверку, то посредник перрывает запрос и воврзащает 401. Если токен валиден, то посредник вызвает метод `c.Next()`, который продолжает обработку запроса.

Этот подход подходит если нужно использовать обработчик на всех обработчиках.

### Группировка запросов в маршрутизации

Чтобы названчить посредника только на некоторые обработчики их нужно сгруппировать с помощью `r.Group()`. После этого можно установить посредников с помощью `r.Use()`:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
)

func main() {
    webApp := fiber.New()

    // Создаем группу с префиксом пути запроса "/authorized".
    authGroup := webApp.Group("/authorized")
    // Добавляем посредника проверки доступа в группу.
    authGroup.Use(accessMiddleware)
    // Добавляем обработчики запросов в группу.
    authGroup.Post("/action/1", func(ctx *fiber.Ctx) error {})
    authGroup.Post("/action/2", func(ctx *fiber.Ctx) error {})
    authGroup.Post("/action/3", func(ctx *fiber.Ctx) error {})

    // Создаем группу с префиксом пути запроса "/public".
    publicGroup := webApp.Group("/public")
    // У группы нет посредников.
    // При запросах к группе сразу выполняются обработчики.
    publicGroup.Post("/action/1", func(ctx *fiber.Ctx) error {})
    publicGroup.Post("/action/2", func(ctx *fiber.Ctx) error {})
    publicGroup.Post("/action/3", func(ctx *fiber.Ctx) error {})


    logrus.Fatal(webApp.Listen(":80"))
}
```

Здесь создаются две группы запросов: `authGroup` и `publicGroup`. В группу `authGroup` добавляется посредник `accessMiddleware`, проверяющий наличие корректного токене в запросе. В группе `publicGroup` посредника не добавляется, поэтому запросы идут сразу к обработчикам.

### Логирование запросов

Чтобы добавить логирование всех запросов в Fiber-приложение, нужно подключить к проекту пакет *github.com/gofiber/fiber/v2/middleware/logger* и инициализировать его перед всеми обработчиками:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/logger"
    "github.com/sirupsen/logrus"
    "time"
)

func main() {
    webApp := fiber.New()

    webApp.Use(logger.New())
    webApp.Get("/", func(c *fiber.Ctx) error {
        // Создаем искусственную задержку, чтобы проверить логирование.
        time.Sleep(300 * time.Millisecond)

        return c.SendString("OK")
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

По умолчанию логирование идёт в консоль, но можно настроить логирование в файл или другие системы логирования.

Также при инициализации посредника для логирования можно указать формат логов. Для этого нужно передать в функцию инициализации посредника параметр `logger.Config`:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/logger"
    "github.com/sirupsen/logrus"
    "time"
)

func main() {
    webApp := fiber.New()

    webApp.Use(logger.New(logger.Config{
        Format:     "${time} ${method} ${path} - ${status} - ${latency}\n",
        TimeFormat: "2006-01-02 15:04:05.000000",
    }))
    webApp.Get("/", func(c *fiber.Ctx) error {
        // Создаем искусственную задержку, чтобы проверить логирование.
        time.Sleep(300 * time.Millisecond)

        return c.SendString("OK")
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

Хорошей практичкой является логирование идентификатора, позволяющего свзяать все логи в рамках одного запроса. Для этого нужно подклчить к проекту пакет `github.com/gofiber/fiber/v2/middleware/requestid` и инициализировать его перед посредником для логирования:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/logger"
    "github.com/gofiber/fiber/v2/middleware/requestid"
    "github.com/sirupsen/logrus"
    "time"
)

func main() {
    webApp := fiber.New()

    webApp.Use(requestid.New())
    webApp.Use(logger.New(logger.Config{
        Format:     "${locals:requestid}: ${time} ${method} ${path} - ${status} - ${latency}\n",
        TimeFormat: "2006-01-02 15:04:05.000000",
    }))
    webApp.Get("/", func(c *fiber.Ctx) error {
        // Создаем искусственную задержку, чтобы проверить логирование.
        time.Sleep(300 * time.Millisecond)

        logrus.WithFields(logrus.Fields{
            "request_id": c.Locals("requestid"),
        }).Warn("something went wrong")

        return c.SendString("OK")
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

### Ограничение количества запросов (throttling)

Для тротлинга используется пакет `github.com/gofiber/fiber/v2/middleware/limiter`:

```go
package main

import (
    "github.com/gofiber/fiber/v2"
    "github.com/gofiber/fiber/v2/middleware/limiter"
    "github.com/sirupsen/logrus"
    "time"
)

func main() {
    webApp := fiber.New()

    webApp.Use(limiter.New(limiter.Config{
        KeyGenerator: func(c *fiber.Ctx) string {
            return c.IP()
        },
        Max:        3,
        Expiration: 10 * time.Second,
    }))
    webApp.Get("/", func(c *fiber.Ctx) error {
        return c.SendString("OK")
    })

    logrus.Fatal(webApp.Listen(":80"))
}
```

Здесь мы ограничиваем доступ 3мя запросами раз в 10 секунд, по достижению лимита будет отправляется ответ с кодом *429*.

В данном примере используется IP-адрес клиента, чтобы определить источник запроса. Но можно использовать любой другой идентификатор, например, идентификатор пользователя или сессии.

### Дополнительно

1. [Fiber Middleware](https://docs.gofiber.io/guide/routing#middleware)
2. [Fiber Request ID Middleware](https://docs.gofiber.io/api/middleware/requestid)
3. [Fiber Logging Middleware](https://docs.gofiber.io/api/middleware/logger)
4. [Fiber Limiter Middleware](https://docs.gofiber.io/api/middleware/limiter)

## JWT-авторизация на сервере

**WT (JSON Web Token)** — это специальный формат токена, который позволяет безопасно передавать данные между клиентом и сервером.

JWT-токен состоит из трех частей, которые разделены точкой:

- **Header** или **заголовок** — информация о токене, тип токена и алгоритм шифрования
- **Payload** или **полезные данные** —  данные, которые мы хотим передать в токене. Например, имя пользователя,  его роль, истекает ли токен. Эти данные представлены в виде JSON-объекта
- **Signature** или **подпись** — подпись токена, которая позволяет проверить, что токен не был изменен

Обычный токен имеет формат:

```
xxxxx.yyyyy.zzzzz
```

Пример токена:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

#### Header

Заголовок обычно состоит из JSON-объекта с двумя свойствами:

* Тип токена, у нас — **JWT**
* Алгоритм шифрования, у нас — **HMAC SHA256**

Далее этот Json-объект хэшируетс с помощью _Base64Url-кодирования_ для представления в виде компактной строки.

У нас заголовок JWT-токена имеет следующие значения:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

#### Payload

Вторая часть токена — полезная нагрузка в виде JSON-объекта, она содержит данные об авторизованном пользователе. Значение будет различно для каждого веб-приложения, сюда можно записать любые публичные данные необходимые для авторизации.

Эта часть также хэшируется с помощью _Base64Url-кодирования_.

У нас для этой части такое значение:

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

Здесь:

* sub — идентификатор клиента;
* name — имя;
* iat — время создания токена

При составлении полей рекоменудуется использовать имена из [документации IANA (Internet Assigned Numbers Authority)](https://www.iana.org/assignments/jwt/jwt.xhtml).

Основная причина сокращения названия полей в полезной нагрузке — это уменьшение размера токена после шифрования.

#### Signature

Для создания подписи нужно взять закодированный заголовок, закодированную полезную нагрузку, секретную строку и зашифровать эти данные. При этом нужно использовать алгоритм шифрования из заголовка JWT-токена.

В примере используется  *HMAC SHA256* и секретная строка `your-256-bit-secret`:

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  your-256-bit-secret
)
```

Подпись используется чтобы проверить, что сообщение не было изменено при передаче. Она также позволяет подтвердить, что JWT-токен является тем, кем он представляется.

#### Собираем все части

В результате генерации получается три Base64-URL-закодированные строки, которые разделены точкой.

Для декодирования, проверки и генерации можно использовать — [jwt.io Debugger](https://jwt.io/)

### Алгоритм работы с JWT-токеном

Процес аутентификации и авторизации с JWT-токеном между веб-браузером и веб-приложением выглядит следующим обрзаом:

1. Веб-браузер отправляет запрос веб-приложению с логином и паролем;
2. Веб-приложение проверяет логин и пароль, если всё ок, то генерируется JWT-токен и отправляет его к веб-браузеру. При генерации JWT-токена веб-приложение ставит подпись секретным ключом, которых хранится только в веб-приложении;
3. Веб-браузер сохраняет JWT-токен и отправляет его вместе с каждым запросом в веб-приложение;
4. Веб-приложение проверяет JWT-токен и если он верный, то выполняет действие от имени авторизованного пользователя.

Безопасность коммуникации между бразуером и приложением заключается в том, что токены генерируются и подписываются только со стороны веб-приложения. Такой токен невозможно подделать без знаяния секретного ключа.

Подпись токена проихсодит с помощью шифрования. С помощью подписи приложение проверяет, что токен действительно был сгенерирован этим приложением. Для шифрования могут исползоваться различные алгоритмы.

### JWT-авторизация в Go

Расмотрим пример реализации атентификации, для этого реализуем:

* Регистрация пользователя;
* Вход в аккаунт — аунтификация;
* Получение информации о своём аккаунт – только для авторизированных пользователей.

#### Регистрация аккаунта

Когда пользователь заходит на веб-сайт, он видит форму регистрации с  тремя полями: имя, электронная почта и пароль. После заполнения формы  пользователь нажимает кнопку «Зарегистрироваться», и веб-браузер  отправляет HTTP-запрос `POST /register` в наше веб-приложение. Мы реализуем обработчик этого HTTP-запроса следующим образом:

```go
package main

import (
    "errors"
    "fmt"
    "github.com/gofiber/fiber/v2"
    "github.com/sirupsen/logrus"
)

func main() {
    app := fiber.New()

    authHandler := &AuthHandler{&AuthStorage{map[string]User{}}}

    app.Post("/register", authHandler.Register)

    logrus.Fatal(app.Listen(":80"))
}

type (
    // Обработчик HTTP-запросов на регистрацию и аутентификацию пользователей
    AuthHandler struct {
        storage *AuthStorage
    }

    // Хранилище зарегистрированных пользователей
    // Данные хранятся в оперативной памяти
    AuthStorage struct {
        users map[string]User
    }

    // Структура данных с информацией о пользователе
    User struct {
        Email    string
        Name     string
        password string
    }
)

//  Структура HTTP-запроса на регистрацию пользователя
type RegisterRequest struct {
    Email    string `json:"email"`
    Name     string `json:"name"`
    Password string `json:"password"`
}

// Обработчик HTTP-запросов на регистрацию пользователя
func (h *AuthHandler) Register(c *fiber.Ctx) error {
    regReq := RegisterRequest{}
    if err := c.BodyParser(&regReq); err != nil {
        return fmt.Errorf("body parser: %w", err)
    }

    // Проверяем, что пользователь с таким email еще не зарегистрирован
    if _, exists := h.storage.users[regReq.Email]; exists {
        return errors.New("the user already exists")
    }

    // Сохраняем в память нового зарегистрированного пользователя
    h.storage.users[regReq.Email] = User{
        Email:    regReq.Email,
        Name:     regReq.Name,
        password: regReq.Password,
    }

    return c.SendStatus(fiber.StatusCreated)
}
```

#### Вход в аккаунт

Когда пользователь заходит на страницу входа в аккаунт, он видит форму с двумя полями: электронная почта и пароль. Эти поля являются учетными  данными пользователя. После заполнения формы пользователь нажимает  кнопку «Войти», и веб-браузер отправляет HTTP-запрос `POST /login` в наше веб-приложение. Обработчик этого запроса будет выглядеть следующим образом:

```go
// Структура HTTP-запроса на вход в аккаунт
type LoginRequest struct {
    Email    string `json:"email"`
    Password string `json:"password"`
}

// Структура HTTP-ответа на вход в аккаунт
// В ответе содержится JWT-токен авторизованного пользователя
type LoginResponse struct {
    AccessToken string `json:"access_token"`
}

var (
    errBadCredentials = errors.New("email or password is incorrect")
)

// Секретный ключ для подписи JWT-токена
// Необходимо хранить в безопасном месте
var jwtSecretKey = []byte("very-secret-key")

// Обработчик HTTP-запросов на вход в аккаунт
func (h *AuthHandler) Login(c *fiber.Ctx) error {
    regReq := LoginRequest{}
    if err := c.BodyParser(&regReq); err != nil {
        return fmt.Errorf("body parser: %w", err)
    }

    // Ищем пользователя в памяти приложения по электронной почте
    user, exists := h.storage.users[regReq.Email]
    // Если пользователь не найден, возвращаем ошибку
    if !exists {
        return errBadCredentials
    }
    // Если пользователь найден, но у него другой пароль, возвращаем ошибку
    if user.password != regReq.Password {
        return errBadCredentials
    }

    // Генерируем JWT-токен для пользователя,
    // который он будет использовать в будущих HTTP-запросах

    // Генерируем полезные данные, которые будут храниться в токене
    payload := jwt.MapClaims{
        "sub":  user.Email,
        "exp":  time.Now().Add(time.Hour * 72).Unix(),
    }

    // Создаем новый JWT-токен и подписываем его по алгоритму HS256
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, payload)

    t, err := token.SignedString(jwtSecretKey)
    if err != nil {
        logrus.WithError(err).Error("JWT token signing")
        return c.SendStatus(fiber.StatusInternalServerError)
    }

    return c.JSON(LoginResponse{AccessToken: t})
}
```

Для генерации JWT-токена, используется библиотека [jwt-go](https://github.com/golang-jwt/jwt). 

В полезной нагрузке JWT-токена записывается электронная почта пользователя как идентификатор, а также время когда токен перестанет быть действительным. В примере это 72 часа.

#### Получение информации об аккаунте

Когда пользователь прошел аутентификацию, он получил JWT-токен,  который будет использовать в последующих HTTP-запросов для авторизации.  Есть разные способы передавать токен в HTTP-запросе. Для этого можно  использовать заголовок *Authorization*, параметр запроса или даже куки веб-браузера. Мы будем использовать заголовок *Authorization*, так как это наиболее распространенный способ передачи JWT-токена в HTTP-запросе.

Когда пользователь заходит на свою страницу, веб-браузер отправляет HTTP-запрос `GET /profile` в наше веб-приложение. Обработчик этого запроса выглядит следующим образом:

```go
package main

import (
    ...
    jwtware "github.com/gofiber/contrib/jwt"
    jwt "github.com/golang-jwt/jwt/v5"
    ...
)

const (
    contextKeyUser = "user"
)

func main() {
    app := fiber.New()

    ...

    // Группа обработчиков, которые требуют авторизации
    authorizedGroup := app.Group("")
    authorizedGroup.Use(jwtware.New(jwtware.Config{
     SigningKey: jwtware.SigningKey{
      Key: jwtSecretKey,
     },
     ContextKey: contextKeyUser,
    }))
    authorizedGroup.Get("/profile", userHandler.Profile)

    logrus.Fatal(app.Listen(":80"))
}

// Структура HTTP-ответа с информацией о пользователе
type ProfileResponse struct {
    Email string `json:"email"`
    Name  string `json:"name"`
}

func jwtPayloadFromRequest(c *fiber.Ctx) (jwt.MapClaims, bool) {
    jwtToken, ok := c.Context().Value(contextKeyUser).(*jwt.Token)
    if !ok {
        logrus.WithFields(logrus.Fields{
            "jwt_token_context_value": c.Context().Value(contextKeyUser),
        }).Error("wrong type of JWT token in context")
        return nil, false
    }

    payload, ok := jwtToken.Claims.(jwt.MapClaims)
    if !ok {
        logrus.WithFields(logrus.Fields{
            "jwt_token_claims": jwtToken.Claims,
        }).Error("wrong type of JWT token claims")
        return nil, false
    }

    return payload, true
}

// Обработчик HTTP-запросов на получение информации о пользователе
func (h *UserHandler) Profile(c *fiber.Ctx) error {
    jwtPayload, ok := jwtPayloadFromRequest(c)
    if !ok {
        return c.SendStatus(fiber.StatusUnauthorized)
    }

    userInfo, ok := h.storage.users[jwtPayload["sub"].(string)]
    if !ok {
        return errors.New("user not found")
    }

    return c.JSON(ProfileResponse{
        Email: userInfo.Email,
        Name:  userInfo.Name,
    })
}
```

Так как проверка авторизации может происходить во многих  HTTP-обработчиках, мы вынесли ее на уровень посредников. В  микрофреймворке Fiber для этого есть готовый посредник — [jwtware](https://github.com/gofiber/jwt).

При инициализации посредника указываются два свойства:

- *SigningKey* — секретный ключ JWT-токена
- *ContextKey* — название поля, по которому хранится объект  JWT-токена авторизованного пользователя. Этот объект можно использовать в любом обработчике группы `authorizedGroup`

Когда приходит HTTP-запрос на получение информации о пользователе, веб-приложение проверяет, что в заголовке *Authorization* указан корректный JWT-токен. Если проверка прошла успешно,  веб-приложение ищет пользователя в хранилище по электронной почте,  которая записана в полезной нагрузке JWT-токена. Если пользователь был  найден, то авторизация пройдена, и мы возвращаем в HTTP-ответе  информацию об этом пользователе.
