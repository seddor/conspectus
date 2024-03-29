# Network mapper (Nmap)

[NMap](https://nmap.org/) — программа для сетевого анализа.

## Определение целей сканирования

Сканирование стоит проводить из под рута (или друго привелигерированного пользоватля).

Доменные имена можно указывать в [CIDR-формате](https://ru.wikipedia.org/wiki/%D0%91%D0%B5%D1%81%D0%BA%D0%BB%D0%B0%D1%81%D1%81%D0%BE%D0%B2%D0%B0%D1%8F_%D0%B0%D0%B4%D1%80%D0%B5%D1%81%D0%B0%D1%86%D0%B8%D1%8F) для сканирования подсетей.

В качестве улей можно указывать адреса и диапозоны протокола ipv4:
```bash
nmap 192.168.7.1
nmap 192.168.1.1/24
nmap 192.168.1.1-25
```
Можно передавать список адрессов в формате файла:
```bash
nmap -iL <список>
```
Для исключения отдельных хостов и подсетей используются опции:
```bash
nmap --exclude --excludefile <список>
```

Для получения дополнильтельной информации используется параметр **-v/-vv/-vvv**. В процессе сканирования можно уменьшать/увеличивать детальность нажимая v/shift+v.

## Обнаружение хостов

Запрос, который позволяет определить имена хостов в заданной сети:
```bash
nmap -sL skillbox.ru/24
```
Для обнаружения работающих хостов используется **-sn**. Также этап обнаружения пропускается опцией **-Pn**, а **-sn** нужна для отключения сканирования портов.

Для отслеивания передвижения пактов исплользуется:
```bash
nmap --traceroute skillbox.ru -sn -Pn
```
В выводе указывается количество узлов, время доступа и сетевые адреса маршрута.

В графической версии nmap — zenmap есть функция визуализации топологии в виде графов.

## Сканирование портов

Для TCP SYN (скрытого) сканирования используется:
```bash
nmap -sS skillbox.ru
```
Иногда для тестирования нужно поное подключени. Для этого используется соединение TCP connect:
```bash
nmap -sT skillbox.ru
```
Отличие заметно лишь при детальном рассмотрении дампов трафика и логов систем обнаружения вторжения.

Некоторые слеужбы используют UDP порты для установления соединения. Для них используется:
```bash
nmap -sU skillbox.ru
```

Другие флаги:
* -sN (TCP Null scan) — устанавливает значение заголовка TCP флага раное 0.
* -sF (FIN scan) — устанавливает только TCP FIN.
* -sX (Xmas scan) — устанавливает флаги FIN, PSH, URG.

Такое сканирование отправляет неправильные пакеты, имитирую ошибка соединения, что иногда позволяет определить скрытые порты.

[Документация](https://nmap.org/book/scan-methods-null-fin-xmas-scan.html)

## Работа с FTP-сервером

**-b** анализирует ответы о неправильной авторизации заданного сервера:
```bash
nmap -b ftpuser:PassW0rd@192.168.0.1:21 10.0.2.1
```
Диапозон сканируемых портов опредлеяется **-p**:
* `nmap -p 80,443 skillbox.ru` — сканирует только 80 и 443 порты.
* `nmap -p80-443 skillbox.ru` — сканирует диапозон портов с 80 по 443.
* `nmap -p- skillbox.ru` — сканирует весь диапозон портов.
Для исключения используется **--exclude-ports**:
```bash
nmap -p21-443 --exclude-ports 80 skillbox.ru
```
Без опций nmap сканирует 1000 самых используемых портов, но это число можно регулировать:
```bash
nmap --top-ports 10 skillbox.ru
```

## Обнаружение версий операционных систем и сервисов

Для обнаружения версий ОС используется **-O**:
```bash
nmap -O -p 25,80,443 skillbox.ru
```
В результате nmap запустит скрипт обнаружения и выдаст наиболее вероятные результаты.

Для обнаружения версий сервисов используется **-sV**:
```bash
nmap -sV skillbox.ru -p25,80,443
```

Сервисы могут специально давать неверные ответы, это бывает редко. Стоит помнить, что запущенные скрипты оставляют много следов для систем обнаружения.

## Производительность и тайминг

Скопрость сканирования можно регулировать, это может быть полезно для избегания обнаружения сканирования, или ускорения сканирования.

Производтельность задаётся в **-T<0-5>**, есть 6 режимов:
* 0 — не используется, т.к. займёт вечность.
* 1 — самый медленный режим, но самый скрытный
* 2 — медленное сканирования для снижения нагрузки
* 3 — "вежливый", используется по дефолту.
* 4 — быстрый и иптимальный
* 5 — самый быстрый

Тайминги — это время задержек и простоев в сканировании. Опция задаёт маскимальое число параллельно запущенных потоков сканирования и диапозон задержки для подаваемых в сеть сигналов.

Определяем   размер   групп   хостов   для   параллельного сканирования: 
```
--min-hostgroup/max-hostgroup <size>
```
Задаём максимальное и минимальное количество запросов для группы хостов:
```
--min-parallelism/max-parallelism <numprobes>
```
Устанавливаем  значение  повторных  попыток  сканирования портов:
```
--max-retries
```
Устанавливаем паузы сканирования между портами:
```
--scan-delay/--max-scan-delay
```

Следует использовать осторожно, т.к. могут существенно замедлить сканирование.

## Обход фаерволов

Для разделения TCP заголовков на несколько пакетов:
```bash
nmap -f skillbox.ru
```

Можно задать размер фрагмента (не мнее 16 байт и кратно 8):
```bash
nmap -f skillbox.ru -mtu 16
```

Для сокрытия источника сканрованния можно использовать метод отправки пакетов, подписанных фиктивными адесами:
```bash
nmap -n -D 192.168.1.1,192.168.1.2,192.168.1.3 192.168.1.100
```
Опция **-n** позволяет не использовать разрешение DNS.

## Спуфинг

Спуфинг отправителя — это подмена источника сканирования. **-S** указывает источник запроса, и все результаты сканирования будут отправлены на него. В запросе следует указать используемый интерфейс опцией **-e**. Опционально: порт отправителя опцией **-g/--source-port** и опция пропуска обраружения хостов **-sn**:
```bash
nmap -S yandex.ru skillbox.ru -e eth0 -g 53 -sn
```

Некоторые сервисы оказывают доверие определённым исходящим портам:
* DNS 53/TCP/UDP
* FTP 20,21,22/TCP
* DHCP 67/UDP
* Kerberos 88/TCP/UDP

Чтобы скрыть местоположение, используется http или socks прокси:
```bash
nmap --proxies socks4://127.0.0.1:9050 skillbox.ru -p80
```
**data-length** добавляет в пакет установленный размер случайных данных:
```bash
nmap --data-length 128 skillbox.ru
```
Вместо случайных данных можно добавить в пакет полезную нагрузку в двоичном коде.
```bash
nmap --data 0xDAEDBEEF skillbox.ru
```

## Скриптовый движок

Получение субдоменов цели:
```bash
nmap --script=dns-brute -Pn -sn skillbox.ru
```
При сканировании некоторых целей можно извлечь доменные имена почтовых, тестовых и прокси-серверов, а также vpn и sip. Последнее наиболее интересно, так как часто они менее защищены и имеют уязвимости.

Если скрпит не даёт результатов, можно проверить его работы в режиме отладки опцией **--script-trace**.

Обновить базу скриптов можно командой **--script-updatedb**.

Некоторые скрипты требуют установки дополнительных модулей, [подробнее](https://nmap.org/nsedoc/).

## Управление выводом

Отображения резлуьтатов в стандартном формате:
```bash
nmap skillbox.ru -oN result
```
Отображение в XML-формате
```bash
nmap skillbox.ru -oX result
```

Наиболее удобным форматом для обработки из терминала является Grepable.

Для формирования отчёта рекомендуется использовать **--webxml**, который улучает портируемость результата сохранённого в XML.

Для вывода информации об оставшемся времени сканирования можно нажать **i**.

Можно ограничить вывод только открытыми портами опцией **--open**. Чтобы узнать почему nmap читает порт открытам или закрытым используется **--reason**.
