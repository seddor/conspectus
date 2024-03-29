# Модульные тесты

## Три закона TDD

1. Не пишите код продукта, пока не напишите отказной модульный тест.
2. Не пешите модульный тест в объеме большем, чем необходимо для отказа. Невозможность компиляции является отказом.
3. Не пишите код продукта в объеме большем, чем необходимо для прихождения текущего отказного теста.

При таком подходе мы пишем тесты на каждый аспект нашего продукта. Громадный объем тестов, сравнимый с объемом кода приложения может повлечь немало организационных проблем.

## О чистоте тестов

* Тесты "на скорую руку" равносильны полному остсутвию тестов, если не хуже;
* Тесты должно изменяться по мере развития продукта;
* Тестовый код не менее важен, чем код продукта.

## тесты как средство обеспечения изменений

* Без тестов утрачивается всё то, что обеспечивает гибкость кода продукта;
* При наличии тестов вносить изменения в код проще, т.к. можно сразу проверить работу остальных частей системы.

## Чистые тесты

* Самая важная характерситика чистоты теста: удобочитемость;
* В тестовом коде необходимо передать максимум информации минимумом выразительных средств;
* Кадлый тест можно раздеить на 3 части: построение, операция, проверка.

## Предметно-ориентированный язык программирования

Для тестов в ситеме можно построить свой набор кода для написания тестов конкретно для системы, вместо использования тестового API напрямую, это позволит упростить написание и понимание будущих тестов.

## Двойной стандарт

Тестовый код должен быть прост, лаконичен и выразителен, но он не обязательно должен быть эффективен как продакшен код.

## Одна проверка на тест

Использование assert должно быть сведено к минимуму (в идеале к одному) в одной тесте, тест должен проверять только что-то одно.

## Одна концепция на тест

В каждой тестовой функции должна тестироваться только одна концепкция.

## F.I.R.S.T.

Чистые тесты должны обладать ещё пятью харакетиристиками:

* Быстрота (Fast) — Тесты должно выполняться быстро. Долго выполняемые тесты не будут запускаться часто из-за чего сложнее выявить проблемы;
* Независимость (Indepent) — Тесты не должны зависеть друг от друга. Один тест не должен создавать условия для выполнения следующего теста. Все тесты должны выполняться независимо, в любом порядке;
* Повторяемость (Repeatable) — Тесты должны давать повторяемые результаты в любой среде;
* Очевидность (Self-Validating) — Результатом выполнение тесты должно быть либо "тест прошёл", либо "тест не прошёл". Т.е. рузультат теста должен быть сразу очевиден;
* Своевременность (Timely) — Тесты должны создаваться своевременно. Модыльные тесты пишутся **непосредственно перед** кодом продукта, обеспечивабщим их прохождение.
