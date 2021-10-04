# Переходы

Для перехода небоходимо:

* Два набора свойства:
    * начальный 
    * конечный
* свойство `transition` — содержит описание свойст и характеристик анимации перехода.
* Инициатор — действие которое вызвает переход(например: `:hover`, `:target`, `:focus`, `:active`, или инициация перехода из JS).

```CSS
.button {
    /* свойство перехода */
    transition-property: transform, backround-color;
    /* длительность перехода */
    transition-duration: .3s, 500ms;
    /* задержка перехода */
    transition-delay: 0s, 0.5s;

    backround-color: #ccc;
}
.button:hover {
    transform: scale(1.2);
    backround-color: #f00;
}
```

Так-же существует короткая запись:
```CSS
.short {
    /* property duration [timing-function] [delay] */
    transition: transform .5s ease-in 1s;
}
```

[Подробнее](https://developer.mozilla.org/ru/docs/Web/CSS/transition)

# Анимация

Свойство `animation`, позволяет анимировать переходы между ключенвыми кадрами.
Для созадния анимции нужно
1. Определить ключенвые кадры — содержат свойства элемента в определённый момент времени.
2. Применить анимацию к элементу.

[Подробнее](https://developer.mozilla.org/ru/docs/Web/CSS/CSS_Animations/Ispolzovanie_CSS_animatciy)
