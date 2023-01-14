Здраствуйте.

Возможные уязвимости:
Для openSSH версии 6.6:
* Username Enumeration:
  * https://www.exploit-db.com/exploits/45233
  * https://www.exploit-db.com/exploits/45210


Сканировал таким образом:
1. Получение возможной версии ОС:
sudo nmap -O scanme.org
В результатате вероятнее всего используется HP P2000 G3 NAS device или Linux.
```
HP P2000 G3 NAS device (93%), Linux 2.6.32 (92%), Linux 2.6.32 - 3.1 (92%), Ubiquiti AirOS 5.5.9 (92%), Ubiquiti Pico Station WAP (AirOS 5.2.6) (92%), Linux 2.6.32 - 3.13 (92%), Linux 3.0 - 3.2 (92%), MikroTik RouterOS 6.36 (91%), Linux 4.0 (91%), Linux 3.7 (91%)
```
По HP P2000 G3 NAS device ничего, нет поэтому ищем уязвимости для linux (плюс у сервисов версии указана Ubuntu, что указывает, на то что может использоваться этот дистрибутив).

2. Получения версий сервисов:
```
sudo nmap -sV scanme.org
```
Используется:
* OpenSSH 6.6.1p1
* Apache httpd 2.4.7
На сайте ищем уязвимости для них.
