# Сканирование портов

## nmap

`nmap -sS -T4 -n <target>`

-sS — sin-scan
-T4 — тайминг

Показывает открыте порты, даёт не точную версию о софите используемом на порте, для более точного анализа нужно использовать параметр `-A`. 
По умолчанию npam ищет не по всем портам, а по популярным(по версии nmap), для указания конкретных портов нужно использовать параметр `-p 1,2,4-10`. Для всех портов: `-p 1-65535`.