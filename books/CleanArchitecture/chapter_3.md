# Принципы дизайна

Принципы **SOLID** определяют, как объеденять функции и структуры данных в классы и как эти классы должны сочетаться между собой. В данном контексте "класс" означает интрумент объеденения функций и данных в группы, а не только класс в ООП.

Цели принципов — создать программные структуры, которые:
* терпимы к изменениям;
* просты и понятны;
* образуют основу для копонентов, которые могут использовать во многих программных системах.

## SOLID

* **SPR** (Single Responsibility Principle) — Принцип единственной ответственности. Заключается в том, что компонент должен иметь только одну причину для изменений, т.е. для каждого класса должно быть только одно назначение, которое он должен выполнять. Принцип касается не только функций и классов, но и проявляется в различных формах на более высоких уровнях. На уровне компонентов он превращается в принцип согласованного изменения (Common Closure Principle), а на архитектурном в принцип оси изменений (Axis of Change);
* **OCP** (Open-Closed Principle) —  Принцип открытости/закрытости. Программные сущности должны быть открыты для расширения, но закрты для изменения. Цель принципа — сделать систему легко расширяемой и обезопасить её от влияния изменений, она достигается делением системы на компоненты и упорядочением их зависимостей в иерархию, защищающую компоненты уровнем выше от изменений в компонентах уровнем ниже;
* **LSP** (Liskov substitution principle) — Принцип подстановки Барбары Лисков. Функции, которые используют базовый тип, должны иметь возможность использовать подтипы базового типа не зная об этом;
* **ISP** (interface segregation principle) — Принцип разделения интерфейсов. Принцип призывает избегать зависимости от всего того, что не используется. Множество интерфесы специализрованные для нужд клиентов лучше, чем один общий интерфейс;
* **DIP** (dependency inversion principle) — Принцип инверсии зависимостей. Код реализующий высокоуровневую политику не должен зависеть от кода, реализующего низкоуровневые детали. Нужно зависить от абстракций, а не от конкретных реализаций.
