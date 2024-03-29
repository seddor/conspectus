# Уадлёные ветки

Удалённые ветки — это сслыки на состояния веток в удалёных репозиториях. Это локальные ветки, которые незльзя перемещать, они перемещаются автоматически при получении измений из удалённых репозиториев.

Имя таких веток строится так: `<имя_удалённого_репозитория>/<имя_ветки>`.

## Отправка изменений

```bash
git push <remote_name> <branch_name>
```
Команда отправит `<branch_name>` в `<branch_name>`, елси нужно залить в другую ветку то нужно исползьовать:
```bash
git push <remote_name> <branch_name>:<another_remote_branch_name>
```

## Создание ветки на основе удалённой

```bash
git ckeckout -b <name> <remote_name>/<branch_name>
```

## Осталеживание веток

Отслеживаемые ветки, это локальные ветки, которые напрямую связаны с удалённой веткой. В таких ветках можно выполянить `git pull` и git будет знать с какого сервера и ветки брать изменения.

Создание локальной ветки с отслеживаением удалённой:
```bash
git checkout -b <remote_name> --track <remte_name>/<branch_name>
```

Настройка локальной ветки на отслеживание:
```bash
git branch -u <remote_orign>/<branch_name>
```

```bash
git branch -vv
```
Отображет список локальных веток с дополнительной информацией, включая то, какая из веток отслеживается.

## Удаление веток на удалённом сервере

```bash
git push <remote_name> :<branch_name>
```
