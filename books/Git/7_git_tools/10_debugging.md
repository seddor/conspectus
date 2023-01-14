# Обнаружение ошибок с помощью Git 

## Анотация файла

Из анотации можно узнать какие коммиты внесли последние измения в файл, кто их сделал и когда
```bash
git blame <file>
```
показывает анотации для файла.
* `-L` — орграничивает вывод, например `-L 12,22` покажет вывод анотации с 12 по 22 строку файла.

Вывод примерно такой:
```
^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 12)  def show(tree = 'master')
^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 13)   command("git show #{tree}")
^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 14)  end
^4832fe2 (Scott Chacon  2008-03-15 10:31:28 -0700 15)
9f6560e4 (Scott Chacon  2008-03-17 21:52:20 -0700 16)  def log(tree = 'master')
79eaf55d (Scott Chacon  2008-04-06 10:15:08 -0700 17)   command("git log #{tree}")
9f6560e4 (Scott Chacon  2008-03-17 21:52:20 -0700 18)  end
9f6560e4 (Scott Chacon  2008-03-17 21:52:20 -0700 19)
42cf2861 (Magnus Chacon 2008-04-13 10:45:01 -0700 20)  def blame(path)
42cf2861 (Magnus Chacon 2008-04-13 10:45:01 -0700 21)   command("git blame #{path}")
42cf2861 (Magnus Chacon 2008-04-13 10:45:01 -0700 22)  end
```
`^` в начале хеша означает что измения были внесены в первом коммите в файл.

* `-C` — git проанализирует аноитруемый файл и попытается вяснить откуда изначально появились фрагменты кода, если они появились после перемещения из другого файла. Например, файл `GITServerHandler.m` раздилил на несколько файлов, один из которых `GITPackUpload.m`, если вызвать на нём `git blame -C GITPackUpload.m`, то можно будет увидеть откуда измначально был взят код:
```
f344f58d GITServerHandler.m (Scott 2009-01-04 141)
f344f58d GITServerHandler.m (Scott 2009-01-04 142) - (void) gatherObjectShasFromC
f344f58d GITServerHandler.m (Scott 2009-01-04 143) {
70befddd GITServerHandler.m (Scott 2009-03-22 144)         //NSLog(@"GATHER COMMI
ad11ac80 GITPackUpload.m    (Scott 2009-03-24 145)
ad11ac80 GITPackUpload.m    (Scott 2009-03-24 146)         NSString *parentSha;
ad11ac80 GITPackUpload.m    (Scott 2009-03-24 147)         GITCommit *commit = [g
ad11ac80 GITPackUpload.m    (Scott 2009-03-24 148)
ad11ac80 GITPackUpload.m    (Scott 2009-03-24 149)         //NSLog(@"GATHER COMMI
ad11ac80 GITPackUpload.m    (Scott 2009-03-24 150)
56ef2caf GITServerHandler.m (Scott 2009-01-05 151)         if(commit) {
56ef2caf GITServerHandler.m (Scott 2009-01-05 152)                 [refDict setOb
56ef2caf GITServerHandler.m (Scott 2009-01-05 153)
```

## Бинарный поиск

Если в каком-то коммите что-то сломалось, и нужно выяснить в каком коммите, то можно использовать бинарный поиск:

1. Нужно запусть процесс поиска:
```bash
git bisect start
```
2. Сообщить git, что текущий коммит сломан:
```bash
git bad
```
3. Указать, последний коммит, в котором всё работало:
```bash
git good <commit>
```

После этого git произведёт выяснит сколько коммитов произошло с момента рабочего коммита и достанет один из середины, на нём можно проверить осталась ли проблема, если да, то она внесена до этого коммита или в нём, если нет, то позже. 
Если проблема не проявляется можно продолить поиск:
```bash
git bisect good
```
Git переключится на следующий серединный коммит между текущим и плохим коммитом, на нём сново можно проверить воспроизведение проблемы. Если проблема проявилось, то:
```bash
git bisect bad
```
Git сообщит информацию о коммите, в котором была внесена ошибка.

Для оконачания поиск:
```bash
git bisect reset
```
Git переключит обратно на то место, откуда начинался поиск.

Этот поиск можно автоматичзировать если есть тест, котопый будет возвращать 0 если ошибки нет, и любое другое число при её наличии:
1. Указываем первыйм аргументом плохой коммит, вторым хороший:
```bash
git bisect start HEAD v1.0
```
2. Запускаем бинарный поиск с использыванием теста:
```bash
git bisect run test-error.sh
```
Дальше git пронализирует коммиты и выведет тот, в котором тест упадёт.
