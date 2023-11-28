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

