# Terraform

[Terraform](https://developer.hashicorp.com/terraform) — инструмент, с помощью которого автоматизируется настройка  серверной инфраструктуры. Он интегрирован со всеми популярными облаками и может развернуть одной кнопкой любые доступные там сервисы от БД и серверов, до CDN и балансировщиков.

Terraform реализует подход инфраструктуры как код.

## Инициализация

Для начало нужно выбрать облако которое будет использоваться. Выбранное обалко нужно найти в списке [провайдеров](https://registry.terraform.io/providers), на странице провайдера можно посмотреть документацию по подключению этого провадйера.

[Доукументация для Yandex.cloud](https://cloud.yandex.ru/docs/tutorials/infrastructure-management/terraform-quickstart)

Затем нужно создать файл _main.tf_ с описанием инфраструктуры, пример для yandex cloud:

```Terraform
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex",
      version = "~> 0.87"
    }
  }
}

provider "yandex" {
  zone = "ru-central1-b"
}
```

После этого необохдимо установить провадера:

```bash
terraform init
```

Затем нужно описать план инфрастуктуры в этом же файле:

```Terraform
resource "yandex_compute_instance" "vm-1" {
  name = "terraform-1"

  resources {
    core_fraction = 20
    cores  = 2
    memory = 1
  }

  boot_disk {
    initialize_params {
      image_id = "fd8emvfmfoaordspe1jr"
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.subnet-1.id
    nat       = true
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_rsa.pub")}"
  }
}

resource "yandex_vpc_network" "network-1" {
  name = "network1"
}

resource "yandex_vpc_subnet" "subnet-1" {
  name           = "subnet1"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.network-1.id
  v4_cidr_blocks = ["192.168.10.0/24"]
}
```

Данный план создаст 1 сервер terraform-1.

Длаее нужно запусть проверку плана:

```bash
terraform validate
```

и если всё ок, команду для запуска применения плана:

```bash
terraform apply
```

На экран будет выведен итоговый план инфрастуруктуры, после чего если всё ок нужно ввести `yes` и план будет применён.

## Архитекутра

Провайдером в Terraform может быть не только облако. Любой сервис или инструмент имеющий подходящий HTTP API, может быть реализован в виде  провайдера. Так через Terraform можно управлять мессенджерами, Docker,  кластером Kubernetes, DNS, настройкой алертов в системах мониторинга и  многим другим.

### Как работает Terraform

Выскороуровнево Terraform работает просто: Terraform берёт описание инфраструктуры в виде кода и на его основе выполняет HTTP-запросы для создания этих ресурсов. Для того чтобы контролировать состояние уже созданных ресурсов используется главная особенность Terraform — контроль состояния уже созданных ресурсов.

После создания ресруса Terraform записывает информацию о нём в _terraform.tfstate_. Информация о состоянии текущей инфраструктуре называется состоянием. Его главная задача в соотношении реального состояния ресурсов с описанным в Terraform.

При изменении ресурса Terraform сравнивает, что было и как должно быть. На основе этого строится план, по которому видно какие действия будут производится Terraform. Затем эти изменения подтверждаются пользователем, Terraform посылает запросы провайдеру и записывает новое состояние в файл состояния.

При построении плана Terraform сверяет состояние описанное в файле состояния и реальное состояние инфрастуктуры и приводит её к описанному в файле состояния, это сделано на случай изменений ресрсов, описанных в Terraform, в обоход Terraform. Такое изменение считается плохой практикой, все изменения для ресрсов описанных через Terraform должны изменяться в нём же.

При этом Terraform взаимодействует только с теми сервисами, что были описаны с его использованием.

[Как применяется конфигурация](https://developer.hashicorp.com/terraform/language/resources/behavior#how-terraform-applies-a-configuration)

[О сосотоянии](https://developer.hashicorp.com/terraform/language/state)

### Изменение и удаление ресурсов

Для удаления ресурса, достаточно удалить его описание из Terraform, а для изменения - достаточно поменять его описание.

С изменением ситация сложнее: некоторые измения могут требовать пересоздание ресурса. Поэтому нужно обращать на это внимание при применении плана, иначе можно потерять ресурсы которые удалять нельзя.

Если же нам требуется именно пересоздать ресрус, то используется команда:

```bash
# <address> - это адрес ресурса в конфигурации
# например hcloud_server.devops-example-app
terraform taint [options] <address>
```

Список всех ресрсов можно получить коммандой:

```bash
terraform state list
```

### Импортирование ресурсов

Terraform позволяет импортировать в него ресрусы которые были созданы без его участия, например чтобы добавить инстанс из Yandex.cloud нужно:

1. Добавить описание этого ресурса в файл описания инфрастуруктуры:

```
resource "<resource_type>" "<resource_name>" {
}
```

2. Запустить команду импорта:

```bash
terraform import <resource_type>.<resource_name> <resource_id>
```

После этого ресурс будет добавлен в файл состояния.

Удалить его оттуда можно командой:

```bash
terraform state rm <resource_type>.<resource_name>
```

Затем его нужно удалить из файла с описанием инфраструктуры.

## Переменные

Переменные в terrafrom позволяют делать настройку инфраструктуры более гибкой.

Переменнные не могут изменяться во время работы terrafrom, но их можно задать динамически перед началом работы.

### Определение

Определять переменные можно в любой части конфигурации, однако terrafrom рекоменует класть их отедльно в файл _variables.tf_, пример его содержания:

```Terraform
variable "do_token" {
  description = "Какое-то описание"
  // Тип значения переменной
  type        = string
  // Значение по умолчанию, которое используется если не задано другое
  default = "какое-то значение по умолчанию"
  // Прячет значение переменной из всех выводов
  // По умолчанию false
  sensitive = true
}
```

Все 4 параметра опциональны, но тип лучше указывать всегда. Значение по умолчанию ставиться только когда имеет смысл.

Если значение по умолчанию не указанно, то Terrafrom будет требовать его перед выполнением.

Есть несколько способов задавания этих значений:

1. Через флаг `-var-file`, при запуске Terrafrom:

```bash
terraform apply -var-file=myfile.tfvars
```

2. Terrafrom автоматически загружает все файлы из директории с именем _terraform.tfvars_ и соответствующие маске _\*.auto.tfvars_. Файлы _\*.tfvars_ содержат присваивание переменных их значения:

```Terraform
do_token       = "dop_v1_cafd2fda20a;sldfjk2o3i4hlk13hj41l2jk341;234"
server_numbers = 2
hcloud_token   = "SD@#32FD"
```

### Типы

На данный момент переменные могут быть 5 типов:

- *string*

  ```Terraform
  variable "vpc_cidr_block" {
    description = "CIDR block for VPC"
    type        = string
    default     = "10.0.0.0/16"
  }
  ```

- *number*

  ```Terraform
  variable "instance_count" {
    description = "Number of instances to provision."
    type        = number
    default     = 2
  }
  ```

- *bool*

  ```Terraform
  variable "enable_vpn_gateway" {
    description = "Enable a VPN gateway in your VPC."
    type        = bool
    default     = false
  }
  ```

- *list*: список элементов одного типа

  ```Terraform
  variable "public_subnet_cidr_blocks" {
    description = "Available cidr blocks for public subnets."
    // В скобках указывается тип элементов списка
    type        = list(string)
    default     = [
      "10.0.1.0/24",
      "10.0.2.0/24",
      "10.0.3.0/24",
    ]
  }
  ```

- *map* или *object*: ассоциативный массив состоящий из пар ключ-значение

  ```Terraform
  variable "resource_tags" {
    description = "Tags to set for all resources"
    type        = map(string)
    default     = {
      project     = "project-alpha",
      environment = "dev"
    }
  }
  ```

[Типы переменных](https://developer.hashicorp.com/terraform/language/expressions/types)

### Использование

Для обращение к переменным импользуется префикс `var.`, например:

```Terraform
provider "hcloud" {
  token = var.hcloud_token
}

resource "digitalocean_droplet" "web2" {
  tags = var.resource_tags
}
```

Переменные можно посдтавлять в любые места, где есть присваивание.

## Зависимости

Terraform позволяет извлекать информацию из описания одних ресрсов и использовать её в других. Для доступа используется синтаксис `<RESOURCE TYPE>.<NAME>.<ATTRIBUTE>`:

```Terraform
// Ключ загружается на Digital Ocean
resource "digitalocean_ssh_key" "somename" {
  name       = "Hexlet Example"
  // Загрузка ключа из файловой системы
  public_key = file("files/somename.pub")
}

resource "digitalocean_ssh_key" "anothername" {
  name       = "Hexlet Example"
  public_key = file("files/anothername.pub")
}

resource "digitalocean_droplet" "web" {
  image    = "ubuntu-18-04-x64"
  name     = "web-1"
  region   = "nyc3"
  size     = "s-1vcpu-1gb"
  // Массив ключей
  ssh_keys = [
    digitalocean_ssh_key.somename.fingerprint,
    digitalocean_ssh_key.anothername.fingerprint
  ]
}
```

В примере выше используется атрибут `fingerprint` ресурса `digitalocean_ssh_key`. Этот атрибут не указан явно, Terraform его получает автоматически на основе значения атрибута `public_key`. Полный список атрибутов ресурса доступен в его документации ресурса у провайдера.

Также можно указывать атрибуты которые будут доступы после создания ресруса (например ip-адрес).

Порядок в котором описываются ресурсы не важен, т.к. Terraform автоматически проанализирует описания и расположит их в правильном порядке при составлении и применении плана.

В некоторые случаях Terraform не может определить правильный порядок действий, в таком случае порядок можно задать явно через аргумент [depends_on](https://developer.hashicorp.com/terraform/language/meta-arguments/depends_on):

```Terraform
// Пример с AWS

resource "aws_s3_bucket" "example" {
  bucket = "hexlet-terraform-getting-started-guide"
  acl    = "private"
}

resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"

  // Всегд массив
  depends_on = [aws_s3_bucket.example]
}
```

