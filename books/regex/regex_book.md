# Конспект по книге `Регулярные выржения`.
Автор книги Джеффри Фридл
Большинство примеров ниже и заметок справедливы для диалекта регулярных выражений используемым в PCRE, а также для языка PHP.

[Здесь можно тестить и тренироваться](https://regex101.com/)

#### Содержание
1. [Знакомство с регулярными выражениями](#chapter1)
  * [grep](#chapter1-grep)
  * [Метасимволы](#chapter1-metachar)
    * [Квантификаторы](#Квантификаторы)
  * [Символьные классы](#chapter1-char-class)
  * [Обратные ссылки](chapter1-reverse-link)
  * [Экранирование](#chapter1-shielding)
2. [Дополнительные примеры](#chapter2)
  * [Модификаторы](#chapter2-mod)   
  * [Просмотр вперед/назад (позиционые проверки)](#chapter2-look)
3. [Регулярные выражения: возможности и диалекты](#chapter3)
  * [Режимы обработки регулярных выражений и поиска совпадений](#chapter3-processing-modes)
  * [Стандартные метасимволы и возможности](#chapter3-standart-feature)
4. [Механика обработки регулярных выражений](#chapter4)
  * [Основы поиска совпадений](#chapter4-basics)
  * [Механизмы регулярных выражений](#chapter4-methods)

### Обозначения применяемые в книге

  * `...` — любое произвольное содержимое
  * `•` — пробельный символ

# <a name="chapter1"></a>	 Глава 1 Знакомство с регулярными выражениями

## <a name="chapter1-grep"></a> grep

`egrep (grep)` - консольная утилита ищущая строки в файле в соответствии с регулярным выржаени.
пример:
```
  grep 'cat' file.txt
```
Найдёт все последовательности символов содержащие шаблон `cat`

grep запущеный с параметром `-i` будет игнорировать регистр в при поиске.


## <a name="chapter1-metachar"></a> Метасимволы


  * `^` — Начало строки
  * `$` — Конец строки
  * `.` — один произвольный символ
  * `|` — символ `или` позволяет обхеденять несколь выражений
  * `()` — скобки обозначают группы, также их нужно использовать для работы с метасиволом `|`, например.
```
gr(e|a)y, если использовать без скобок, то получится: "gre" ИЛИ "ay"
```
  * `\<` и `\>` - соответствено начало и конец слова, поддерживается в некотрорых версиях grep.
Началом слова сичтается начало алфовитно-цифровой последовательности, а концом — её конец.
  * `\t` — табуляция
  * `\n` — новая строка
  * `\r` — возврат каретки
  * `\s` — "пробельные" символы, включает в себя проблелы, табуляцию, символы новой строки и возврат коретки.
  * `\S` — инверсия `\s`
  * `\w` — [a-zA-Z0-9_]
  * `\W` — инверсия `\w`
  * `\d` — [0-9]
  * `\D` — инверсия `\d`
  * `\b` — граница слова

## <a name="chapter1-meta-quantifiers"></a> Квантификаторы

  * `?` — означает, что предшевствующий ему сивол является не обязательный, помещается сразу за символом, например `a?` — наличие литерала `a` не обязательно. Также можно применять к группам.
  * `+` — один или несколько повторений предшествующего элемента
  * `*` — любое количество предшествующего элемента, в том числе и нулевое
  * `{min, max}` — задаёт допустимый диапозон количества элементов:
```
a{5,10} — литера "a" от 5 до 10 штук
a{2,} — литера "a" от 2х штук
```

## <a name="chapter1-char-class"></a> Символьные классы

Конструкция `[...]` в ней можно перечислить набор каких-либо символов, которые будут находится в данной позиции теста, например `[a]` - обозначеет только `a`, а `[ea]` — `a` или 'e'.
Пример, нужно найти последовтельность `grey` и `gray`, тогда мжоно использовать шаблон:
```
  gr[ae]y
```
В контексте символьных классов метасимвол `-` обозначает интервал символов, например:
```
[123456] == [1-6]
[a-z] == всему английскому алфовиту в ловер кейсе
[а-яА-Я] == почти всем буквам русского алфавита в любом кейсе(почти, потому что "ё" не входит в интервал, чтобы её включить нужно использовать класс [a-яА-ЯёЁ]
[0-9a-fA-F] == [0123456789abcdefABCDEF]
```
Примечание: `-` является метасимволом только внутри символьного класса, в остальных случаев это обычный символ дефиса, также в начале символьного класса `-` тоже интерпретируется как обычный дефис.

### Инвертированые символьные классы

Конструкия `[^...]` является инверсией символьного класса, т.е. `[^1-6]` совпадают все символы не вхоядщие в интервал 1-6. `^` является метасимволом только если стоит в начале символьного класса, иначе это просто литерал равный `^`.

## <a name="chapter1-reverse-link"></a> Обратные ссылки

Обратные ссылки позволет искать текст, который совпадает с предующей частью выражения, причём текст не известен на момент написания выражения. Например, нужно найти повторяющиеся идущие друг за другом слова:
```
([A-Za-z]+)•+\1•
```
`\1` — ссылается на текст, который совпал с первой группой, соотвествено `\2`,`\3`,`\4` будут указывать на последующие группы

### Сохранение обратных ссылок в переменные

В PHP найденые группы обычно можно передать в специальный массив, из которого потом можно извлечь эти группы, например у [preg_match](http://php.net/manual/ru/function.preg-match.php) третим параметром передаётся ссылка на массив, куда будут записаны найденые группы. (в элементе с индексом 0 будет хравяться вся найденая строка, в остальных группы в соотвеии с порядком групп) для того чтобы не сохранять группу в регулрном выражении при объявлении группы нужно использовать следующий синтаксим: `(?:...)`.


## <a name="chapter1-shielding"></a> Экранирование

Для экранирования используется `\`

# <a name="chapter2"></a>	 Глава 2 Дополнительные примеры

## <a name="chapter2-mod"></a> Модификаторы

Модификаторы в PRCE записываются после регулярного выражения: `/ba[rz]/iu`
здесь i и u — модификаторы

  * `i` — поиск без учета регистра.
  * `g` — не завершать поиск, после нахождения первого совпадения
  * `m` — расширеный режим привязки, в котором метасимволы начала и конца строки совпадают с началом логических строк(которые заверащются `\n` или `\rn`)/
  * `x` — разрешает многострочные ругулярки:
```
\b
#так можно писать комментарии
(
  \w[-.\w]*
  \@
  [-a-z0-9]
)
\b
```

## <a name="chapter2-look"></a> Просмотр вперед/назад (позиционые проверки)

  * `(?=...)` — просмотр вперёд проверяет, что последоввтельность имеет совападения справа
  * `(?<=...)` — просмотр назад, ищет последовательности слева

Оба вида просмотра не "поглощают" найденый текст. Например:
Если задвна регулярка /Jeffrey/ то совпадющий фрагмент будет следующим:

by `Jeffrey` Freddl

Если же использовать просмотр /(?=Jeffrey)/, то результат будет таким:

by ` `Jeffrey Freddl

Т.е. последовательность используется для поиска, но не включает в результат, а находит только позицию начала совпадения

### Инверсионый просмотр (негативные проверки)

Для позиционых проверок существует также инверстированая версия, которая обозначет, что последовательно `не`должна совпадать слева/справа

  * `(?<!...)` — последовательность не может совпадать слева, инверсия для `(?<=...)`.
  * `(?!...)` — последовательность не может совпадать справа, инверсия для `(?=...)`.

# <a name="chapter3"></a> Регулярные выражения: возможности и диалекты

## <a name="chapter3-processing-modes"></a> Режимы обработки регулярных выражений и поиска совпадений

Помимо [глобальных модификаторов](#chapter2-mod) некоторые диалекты реглярок поддерживают локальные их локальные версии, записываемые так:

  * `(?i)` — включение поиска без учета регистра в группе, (в некоторых диалектах, в том числе том, что в PHP будет `(?i:)`)
  * `(?i-)` — выключение поиска без учета регистра, (для PHP: `(?i-:)`)

При использовании поиска без учета регистра в юникоде стоит учитывать особенности языков поиска:

  * не все алфавиты предусматривают деление символов по регистрам
  * в некоторых языках бывает особый *титульный регистр* для букв в начале слов
  * некоторые буквы не имеют однозначного символа другого регистра, например греческая Σ имеет сразу два символа нижнего регистра ζ и σ
  * иногда символ в одной регистре в другом предоставляется несколькими буквами, например ß в другом регистре будет SS
  * некоторые юникодные символа, например, ǰ не имеют односимвольного представления в верхнем регистре, его заглавная версия представляется комбинацией U+006A и U+030C, существуют такие же схемы и для комбинаций из 3х символов

### Литеральный режим

Режим в котором игнорируются почти все метасимволы, например `[a-z]` будет не символьный классом а буквально строкой `[a-z]`. В PHP чтобы включить режим нужно использовать конструкцию `\Q...\E`, в которой игнорируются все метасимволы.


## <a name="chapter3-standart-feature"></a> Стандартные метасимволы и возможности

### Юникод и метасимвол одного любого символа

Поскольку в юникоде некоторые символы являются комбинацией нескольких символов, то `.` может не всегда работать корректно, в PHP для юникодного одного символа существует метасимвол `\X`

### Юникодные свойства

Помимо самих символов юникод так-же предоставляет множество внутренней информации о символе, его тип, алфавит и т.д. Для поиска литерала по свойству юникода используется `\p{...}`, вместо `...`, записывается, свойство, например
  * `\p{L}` — `p{Letter}` — символы, считающиеся буквами
  * `\p{Z}` — разделители
  * `\p{N}` — цифры

Помимо свойств есть подсвойства, такие как регистр буквы. Они обознаются дополнительным символом в свойстве

  * `\p{Ll}` — буквы верхнего регистра
  * `\p{Lu}` — буквы нижнего регистра

[Больше про юникод в PCRE](http://php.net/manual/ru/regexp.reference.unicode.php)


### Операции с подмножествами

Некоторые диалекты регулярок поддерживают операции над множествами, например

  * в .NET `[[a-z]-[aeiou]]` это символьный класс `a-z` без `aeiou`
  * в Jave `[[a-z]&&[^aeiou]]`
  * В PHP нет операций над множествами, но можно использовать `(?![aeiou])[a-z]` чтобы достичь схожего результата

### Групповые выражения в POSIX

В символьных классах можно также использовать групповые выражения POSIX

  * `[[:alun:]]` — алфавитные и цифровые символы
  * `[[:alpha:]]` — алфавитные символы
  * `[[:blank:]]` — пробел и табуляция
  * `[[:cntrl:]]` — управляющие символы
  * `[[:digit:]]` — цифры
  * `[[:graph:]]` — отображаемые символы (не пробелы, не управляющие символы и т. д.)
  * `[[:lower:]]` — алфавитные символы нижнего регистра
  * `[[:print:]]` — аналог [:graph:], но включает пробел
  * `[[:punct:]]` — знаки препинания
  * `[[:space:]]` — все пропуски ([:blank:], символ новой строки, возврат курсора и т. д.)
  * `[[:upper:]]` — алфавитные символы верхнего регистра
  * `[[:xdlgit:]]` — цифры, допустимые в шестнадцатеричных числах (т. е. 0-9a-fA-F)

### Якорные метасимволы

  * `^`, `\A` — метасимвол `^` совпадает с началом текста, или при расшерненом режиме привязки(модификатор m) с новыми строками. `\A` сопадает только с началом
  * `$`, `\Z`, `\z` — конец физической, логической строки. `$` — совпадает с концом текста, преда заверщающим символом строки. `\z` и `\Z` — конец всего текста, на самом деле с концепцией конца строки не всё однозначно, и $ в разным диалектах и режимах может работать по разному, так-же и `\Z`.
  * `\G` — совпадает с позицией в которой завершилось предыдущее найденое соответствие(или с началом строки, если совпадений нет)
  * `\b`, `\B`, `\<`, `\>` — границы слов,

### Группировки

  * Именованное сохранение, группе сохраняемых скобок можно дать имя с помощью (?<имя>...), например: `(?<foo>foo)`, в PHP эта группа попадёт в массив совпадений в элемент с соответвующем индексом *foo*


### Условные конструкции (?if then | else)

Записываются так `(?(condition)yes-pattern|no-pattern)`, пример
```
/(?(?<=foo)bar|baz)/g
```
Означает, что совпадениями будут считаться только *bar* расположенные сразу после *foo*, иначе совпадением будет *baz* не расположенный после *foo*.

# <a name="chapter4"></a> Механика обработки регулярных выражений

## <a name="chapter4-basics"></a> Основы поиска совпадений

Два униврсальный правила для всех типо механизмов работы регялрок

1. Предпочтение отдется более раннему совпадению
2. Стандартные квантификаторы(`*`, `+`, `?` и `{m,n}`) работают масимально. (т.е. пытается найти максимально возможное число совпадений)

## <a name="chapter4-methods"></a> Механизмы регулярных выражний

Существует два механизма работы регулярных выражений, остальные виды являются их комбинацией.

* ДКА — детерменированный конечный автомат
* НКА — недетерменированный конечный автома, используется в большинстве языков программирования

НКА работает так, что паттерн регулярного выражения посивольно сравнивается с текстом, пока не найдёт совпадение первого символа паттерна в тексте, затем проверяется подзходит ли последующий символ паттерна и текста и т.д.

ДКА работает сразу с несколькими потенициальными совпадениями, и отбрасывает неудачные. Основной принцип его работы в том, что он рассматривает все подвыражения и компоненты, и при выборе между двумя равноправными вариантами выбирает один и запоминает другой, что позднее вернутся к нему в случае необходимости.

### Сравнение

#### Предварительная компиляция

Оба типа перед началом поиска совпадений компилируют регулярное выражение во внутренее представление. В НКА компиляция обычно быстрее и с меньшими затратами памяти.

#### Скорость поиска

* В обычных ситуация скорость у обоих типов схожа
* Скорость в ДКА не зависит от конкретного регулярного выражения
* В НКА скорость напрямую зависит от выражения
* НКА проверяет все возможноные комбинации регулярного выражения, и может остановится при нахождении сопадения
* POSIX НКА не остановится, пока не найдёт наибольшое по длине совпадение
* У ДКА меньше возможностей для оптимизации, т.к. сам по себе этот механизм оптимальней, оптимизация сводится к уменьшению работы на этапе компиляции, современные ДКА также оптимизируются за счёт отлаженой компиляции некоторых частей регулярного выражения, однако при этом возникает ситуация. когда ДКА скорость ДКА начинает зависеть от регулярного выражения

#### Совпадаюищй текст

* ДКА и POSIX НКА всегда находит самое длинное совпадение близкое к левому краю
* НКА находят первое совпадение

#### Возможности 


## <a name="chapter4-atomic-group"></a> Атомарная группировка

Атомарная группировка запрещает просмотр назад для найденой части шаблона содержащей эту группировку, с её помощью можно решить прабемы жадности квантификаторов, например паттерн:

```
(\.\d\d([1-9]?)\d+
```
Для поиска дробной части состоящей из 2х цифр [0-9], одной не обязательной [1-9] и одной или больше [0-9]. Из-за особеностей работы квантификаторов, это паттерн найдет совпадение в строке `27.625`, найдёт совпадение в строке но в группу 1 будет записано не `.625` а `.62`, 5 будет отброшена из группы т.к. не является обязательной и будет отдана `\d+` находящемуся за пределами группы.

С атомарной группировкой
```
(\.\d\d(?>[1-9]?))\d+
```  
В группу один попадёт ожидаемое совпадение `.625`

Атомарные группы запрещают поиск назад, если часть шаблона найдена, поэтому с ней нужно быть осторожным

### Оптимизация с помощью атомарной группировки

Для выражения `^\w+:` в строке `Subject` отсутвует `:` и поиск завершиться неудачей, однако, чтобы понять это механизм обработки вражения сначало пройдёт по всей строке(`\w+` совпадает для каждого символа строки), затем механизм перейдёт к `:` и будет возврщатся к предыдущему состоянию, пока не вернётся к началу. Если заключить выражения в атомарную группировку: `^(?>\w+)` то проверка завершится сразу посел не нахождения `:`. В современных реализацияция регулярок, такая оптимизаци может применятся автоматически.

## Захватывающие квантификаторы

Захватывающие квантификаторы сходны с максимальными, но в отличии от них, они не возвращают, то, что уже включили в себя. Например `+` создаёт сохранённые состояния, в то время как `++` их не создаст.

Захватывающие квантификаторы тесно связаны с атомарной группировкой, конструкция `^\w++` аналогична `^(?>\w+)`. Выражения `(\.\d\d(?>[1-9]?))\d+` — с атомарной группировкой, с захвтывающим квантификтором — `(\.\d\d[1-9]?+)\d+`

## Максимальна ли конструкция выбора

Констракция выбора `(foo|bar)` в большинства систем с НКА механизмом регулярок работает по упорядоченному принципу, т.е. при нахождении совпадения дальнейший поиск альтернатив не производится, в остальных случаях(ДКА, POSIX НКА) будет выбран максимальный по длине совпадающий вариант.

