# Подпись 

Git позволяет использование GPG-подписей.

## Генерация ключа

Ключ генерируется командой:
```bash
gpg --gen-key
```

Посмотреть существующие ключи:
```bash
gpg --list-keys
```

Настроить git для исползьования ключа:
```bash
git config --global user.signkey <key>
```

## Подпись тегов

Подписи тегов:
```bash
git tag -s v1.5
```

Подпись будет видно если сделать для тега `git show <tag>`

```
tag v1.5
Tagger: Ben Straub <ben@straub.cc>
Date:   Sat May 3 20:29:41 2014 -0700

my signed 1.5 tag
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v1

iQEcBAABAgAGBQJTZbQlAAoJEF0+sviABDDrZbQH/09PfE51KPVPlanr6q1v4/Ut
LQxfojUWiLQdg2ESJItkcuweYg+kc3HCyFejeDIBw9dpXt00rY26p05qrpnG+85b
hM1/PswpPLuBSr+oCIDj5GMC2r2iEKsfv2fJbNW8iWAXVLoWZRF8B0MfqX/YTMbm
ecorc4iXzQu7tupRihslbNkfvfciMnSDeSvzCpWAHl7h8Wj6hhqePmLm9lAYqnKp
8S5B/1SSQuEAjRZgI4IexpZoeKGVDptPHxLLS38fozsyi0QyDyzEgJxcJQVMXxVi
RUysgqjcpT8+iQM1PblGfHR4XAhuOqN5Fx06PSaFZhqvWFezJ28/CLyX5q+oIVk=
=EFTF
-----END PGP SIGNATURE-----

commit ca82a6dff817ec66f44342007202690a93763949
Author: Scott Chacon <schacon@gee-mail.com>
Date:   Mon Mar 17 21:52:11 2008 -0700

    changed the version number
```

## Проверка тегов

```bash
git tag -v <tag_name>
```
Команда использует GPG для проверки подписи. Чтобы это работало, нужно чтобы публичный ключ автора присутвовал в хранилище ключей.

При отсутвии ключа ответ будет примерно такой:
```
gpg: Signature made Wed Sep 13 02:08:25 2006 PDT using DSA key ID F3119B9A
gpg: Can't check signature: public key not found
error: could not verify the tag 'v1.4.2.1'
```

## Подпись коммитов

Начиная с git 1.7.9 можно подписывать отдельные коммиты.

```bash
git commit -S
```

Для просмотра подписи в логе:
```bash
git log --show-signature
```
>Также вы можете, используя формат с %G?, настроить git log так, чтобы он проверял и отображал любую обнаруженную подпись.
```
$ git log --pretty="format:%h %G? %aN  %s"

5c3386c G Scott Chacon  signed commit
ca82a6d N Scott Chacon  changed the version number
085bb3b N Scott Chacon  removed unnecessary test code
a11bef0 N Scott Chacon  first commit
```
>В данном примере видно, что только последний коммит корректно подписан, а все предыдущие нет.

С git 1.8.3 `git merge` и `git pull` можно использовать с опцией `--verify-signatures`, при её использовании происходит проверка подписи, и в случае если доверной подписи не обнаружено, то слиянение отклоняется.

```bash
git merge -S
```
В этом случае результат мерджа будет подписан.