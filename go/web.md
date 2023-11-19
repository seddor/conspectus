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