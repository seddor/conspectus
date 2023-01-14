# Сбор информации

## Google dorks

Дорк — специализированый запрос на поиск уязвимой информации.

Пример запроса, поиск pdf-документов на сайте:
```
filetype:pdf site:skillbox.ru
```

Некоторые операторы:
* allintext — ищет совпадения в содержании
* inurl — поиск в URL
* - — удаялет ненужные результаты, например: `-github -gitlab`

[Список дорков](https://www.exploit-db.com/google-hacking-database)
[Операторы Яндекса](https://yandex.ru/support/search/query-language/search-operators.html)

## Shodan

[Shodan](https://www.shodan.io/)

[Библиотека Shodan звапросов](https://github.com/jakejarvis/awesome-shodan-queries) — эти дорки помогут найти информацию по оборудованию (веб-камеры, промышленное оборудование и т.д.)


### Синтаксис

* city: Поиск устройств по заданному городу
* country: Поиск по странам
* geo: Поиск по координатам
* hostname: Поиск по названию хоста
* net: Поиск по IP или /x CIDR
* os: Поиск по операционным системам
* port: Поиск по открытым портам
* before/after: Поиск с указанием времени

## Censys

[Censys](https://www.censys.io/) поддерживает более продвинутые фильтры, лучше работает с сертификатами.

## Дополнительные средства поиска

[Wayback Machine](https://web.archive.org/) — с помощью него можно находить слепки страниц
[2ip.ru](https://2ip.ru/) — собирает информацию об интернет ресурсах и пользователях
[Viewdns.info](https://viewdns.info/) — собирает информацию о DNS серверах
[iknowwhatyoudownload](https://iknowwhatyoudownload.com/ru/peer/)
[OSINT Framework](https://osintframework.com/) — база данных различных утилит и сервисов, посвященных разведке открытых источников
