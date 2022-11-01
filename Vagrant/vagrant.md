# Vagrant

Инструмент для создания среды разработки на основе виртуалок. 
Для работы нужен virtualBox или другая среда виртуализации, так-же есть поддержка docker.

Готовые окружения можно найти [здесь](https://app.vagrantup.com/boxes/search)

Для создания окржения нужно в директории проекта выполнить команду:
```bash
vagrant init ubuntu/bionic64
```
команда скачает окружение и создаст в директории файл `Vagrantfile`. Файл содержит описание конфига для окружения на языке Ruby. 

## Конфигурация

Проброс портов:
```ruby
config.vm.network "forwarded_port", guest: 80, host: 8080
```
* guest — хост в виртуальном окружении
* host — порт на рабочей машине

Использование статичного ip
```ruby
config.vm.network "private_network", ip: "192.168.33.10"
```

Для того чтобы применить настройки в запущеном окружении:
```bash
vagrant reload
```

Для настроек самого окружения, установки окружения поддерживаются различные системы развёртки, например `ansible`.

[Документации](https://www.vagrantup.com/docs/provisioning/)

Для запуска конфигурации:
```bash
vagrant provision
```

## Синхронизируемые директории

По умолчанию вагран синхронизирует директорию в которой лежит вагрантфайл, т.е. корень проекта, в виртуалке директорию будет лежать в `/vagrant`, владельцем директории в виртуалке является vagrant. 
Изменить это можно так:

```ruby
onfig.vm.synced_folder ".", "/vagrant", owner: "www-data",
  group: "vagrant"
```

## Работа с vagrant

### Запуск

```bash
vagrant up
```
Запустит виртуальное окржение.

### Подключение

```bash
vagrant
```
Подключится к окружению. 

По умолчанию в окружении используется пользователь vagrant.

Код проекта будет содержатся в директории `/vagrant` виртуалки.

### Остановка

```bash
vagrant halt
```
Остановит виртуалку.

```bash
vagrant reload
```
Полностью удалит всё в вирталке и создаст её заново.


## 10.0.2.2 

Из виртуального окружения хост доступен по адресу 10.0.2.2 