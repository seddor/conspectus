# Шрифты

Файл шрифта — это файл, содержащий набор символов и соответствующих им кодов.

# Размер шрифта

Размер шрифта устанавливается свойством `font-size`. 

Размер шрифта устанавливается с помощью:
* Ключевых слов:
    * абсолютные, зависят от настроек браузера, обычно их не используют:
        * xx-small
        * x-small
        * small
        * medium
        * large
        * x-large
        * xx-large
    * относительные, изменяют размер относительно родителя примерно на 20%, тоже не используются:
        * smaller
        * larger
* Единиц измерения длины:
    * %
    * em — размер в зависимости от родителя
    * rem — размер зависит от корневого элемента
    * px - пиксели
    * pt, pc, ch, ex
    * cm, mm, q, in
    * vh, vw, vmin, vmax

[подробнее про font-size](https://developer.mozilla.org/ru/docs/Web/CSS/font-size)
[про единицы измерения длины](https://developer.mozilla.org/ru/docs/Web/CSS/%D1%80%D0%B0%D0%B7%D0%BC%D0%B5%D1%80)

## Высота строки

Устанавливается двумя значениями:
* Единицы измерения длины
* Множителем, от текущего размера текущего шрифта

[Подробнее](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height)

## Растояние между буквами

Для измения расстояния используется свойство `letter-spacing`, оно принимает в качестве значения любую единицу измерения длины, кроме процентов. По умолчанию знаения равно `normal`.
```CSS
.zero {
    letter-pacing: -2px;
}
```
[Подробнее](https://developer.mozilla.org/en-US/docs/Web/CSS/letter-spacing)

## Растояние между словами

Свойство `word-spacing`. Аналогичное свойству `letter-spacing`. 

[Подробнее](https://developer.mozilla.org/en-US/docs/Web/CSS/word-spacing)

# Начертания шрифта

## Капитель

Начертание, в котором строчные знаки выглядят как уменьшенные прописные. Устанавливается с помощью `font-variant`.
```CSS
h3 {
    font-variant: small-caps;
}
```

[Подробнее](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant)

## Наклон

управляется через `font-style`. Имеет значения:
* normal — прямое, обычный текст
* oblique — наклонные
* italic — курсивные

[Подробнее](https://developer.mozilla.org/en-US/docs/Web/CSS/font-style)

## Насыщеность

управляется через `font-weight`. Имеет значения:
* normal — прямое, обычный текст
* bold — жирный
* light — тонкий

Также можно установить числовое значения от 100 до 900 с шагом сто, значения меняются от маленького к большому

Также существуют относительные значения, зависящее от родителя, измения проиходит на ~300
* lighter — тоньше
* bolder — жирнее 

[Подробнее](https://developer.mozilla.org/en-US/docs/Web/CSS/font-weight)

## Плотность

управляется через `font-stretch`. Имеет значения:
* ultra-condensed
* extra-condensed
* condensed
* semi-condensed
* normal
* semi-expanded
* expanded
* extra-expanded
* ultra-expanded

Не поддерживается safari

[Подробнее](https://developer.mozilla.org/en-US/docs/Web/CSS/font-stretch)

## Семейства шрифтов

Класификация шрифтов в веб:
* с засечками — serif
* без засечек — sans-serif
* рукописный — cursive
* декоративный — fantasy
* моноширинные — monospace

Существует ряд шрифтов которые присутвуют практически в каждой ОС, таким образом их использование считается наиболее безопасным.

Использование шрифта:
```CSS
body {
    font-family: 'Arial';
}
```

Можно записывать несколько шрифтов через запятую, тогда в случае не нахождении первого в списке шрифта будет использован слудующий и так далее пока не будет найден шрифт, иначе будет использоваться шрифт из настроек браузера. Так-же в качестве шрифта можно указть тип шрифта, тогда будет использован шрифт из бразерной настройки с таким свойством:
```CSS
body {
    font-family: 'Arial, Helvetice, sans-serif';
}
```

[Подробнее](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family)


## Нестандартные шрифты

Подключить кастомные шрифты можно так:
```CSS
@font-family {
    font-family: myCustomFont; /* название шрифта, которе будет использоватся в коде */
    src: url(myCustomFont.ttf); /* путь к шрифту */
}
```
Подключение шрифта:
```CSS
body {
    font-family: myCustomFont;
}
```
Если есть вероятность, что какой-либо шрифт уже установлен на пользовательским компутере, можно установить через `local()`.

Можно указать диапозон используемых символов в шрифте:
```CSS
unicode-range: U+000-5FF; /* только латинские символы */
```
Однако `unicode-range`, плохо поддерживается.

Можно привязать файл к определённому начертанию с помощью свойств `font-weight` и `font-style`:
```CSS
@font-face {
    font-family: My Font;
    src: url(MyFont.ttf);
    font-style: normal;
}

@font-face {
    font-family: My Font;
    src: url(MyFontItalic.ttf);
    font-style: italic;
}
```
Далее в коде можно работать с этим шрифтом как со стандартными.

[Подробнее](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face)
[Генератор шрифтов](https://www.fontsquirrel.com/tools/webfont-generator)

## Алгоритм работы браузера со шрифтами

1. Браузер получает и разбирает CSS
2. Встречает в CSS `@font-face`, но **не** скачивает файл шрифт.
3. Разбирает CSS дальше и встречает указание шрифта в `font-family`.
4. Начинает загрузку шрифта.
5. По окончанию загрузки разбирает шрифт и применяет к странице.

## Оптимизация загрузки шрифта

1. Запись шрифта напрямую в CSS в формате base64, подходит для небольших шрифтов.
2. Исключение ненужны символов.
3. Архивация gzip на стороне сервера.
4. Кеширование.
5. Испрользование формата WOFF 2.0
6. Поднятие `@font-face` и его использование в `font-family` выше в коде.

## `font`

Свойство `font` позволяет управлять большинством того, что связано со шрифтом:
```CSS
body {
    font: variant style weight size line-height family
}
```
Обязательные здесь `size` и `family`.

[Подробнее](https://developer.mozilla.org/en-US/docs/Web/CSS/font)
