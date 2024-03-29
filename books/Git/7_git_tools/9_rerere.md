# Rerere

rerere (“reuse recorded resolution” (“повторное использование сохраненных разрешений конфликтов”) — позволяет попросить git запомнить то, как был разрешён некоторую часть конфликта, так что в случае возникновения такого же конфиликта git его разрешает автоматически.

>Существует несколько ситуаций, в которых данная функциональность может быть действительно удобна. Один из примеров, упомянутый в документации, состоит в том, чтобы обеспечить в будущем простоту слияния некоторой долгоживущей ветки, не создавая при этом набор промежуточных коммитов слияния. При использовании rerere вы можете время от времени выполнять слияния, разрешать конфликты, а затем откатывать слияния. Если делать это постоянно, то итоговое слияние должно пройти легко, так как rerere сможет разрешить все конфликты автоматически.
>Такая же тактика может быть использована, если вы хотите сохранить ветку легко перебазируемой, то есть вы не хотите сталкиваться с одними и теми же конфликтами каждый раз при перебазировании. Или, например, вы решили ветку, которую уже сливали и разрешали при этом некоторые конфликты, вместо слияния перебазировать – наврядли вы захотите разрешать те же конфликты еще раз.
>Другая ситуация возникает, когда вы изредка сливаете несколько веток, относящихся к еще разрабатываемым задачам, в одну тестовую ветку, как это часто делается в проекте самого Git. Если тесты завершатся неудачей, вы можете откатить все слияния и повторить их, исключив из них ветку, которая поломала тесты, при этом не разрешая конфликты снова.

Включить rerere:
```bash
git config --global rerere.enabled true
```
Также можно выключить её, создав каталог `.git/rr-cache` в нужном репозитории, для того чтобы включить локально.

При возникновении конфликта можно выполнить команду:
```bash
git rerere status
```
Она покажет для каких файлов rerere сохранил снимки состояний, в котором они были до начала слияния.

```bash
git rerere diff 
```
Показывает текущее состяние разрешения конликта, то с чего начинали решать конфликт, и как его решли.
Можно выбрать нужный вариант развершения.

Тогда в следующий раз, когда git увидит такой же конфликт в этом файле, он сам выбирет нужный вариант.
Если при этом нужно откатить файл до конфликтного состояния, то можно использовать:
```bash
git checkout --conflict=merge <file>
```

Для повторного разрешения конликта автоматически:
```bash
git rerere
```
