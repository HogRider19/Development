Паттерн представляет определенный способ построения программного кода для решения часто встречающихся проблем проектирования. Предполагается, что есть некоторый набор общих формализованных проблем, которые довольно часто встречаются, и паттерны предоставляют ряд принципов их решения.

Существует множество различных паттернов, которые решают разные проблемы и выполняют различные задачи. Но по своему действию их можно объединить в ряд групп. Рассмотрим некоторые группы паттернов. В основу классификации основных паттернов положена цель или задачи, которые определенный паттерн выполняет.

---

При проектировании отношений между классами надо учитывать некоторые общие рекомендации. В частности, вместо наследования следует предпочитать композицию. При наследовании весь функционал класса-наследника жестко определен на этапе компиляции. И во время выполнения программы мы не можем его переопределить.

А класс-наследник не всегда может переопределить код, который определен в родительском классе. Композиция же позволяет динамически определять поведение объекта во время выполнения, и поэтому такой вид связи является более гибким.

Вместо композиции следует предпочитать агрегацию, как более гибкий способ связи компонентов. В то же время не всегда агрегация уместна. Например, у нас есть класс человека, который содержит объект нервной системы. Понятно, что в реальности, невозможно вовне определить нервную систему и внедрить ее в человека. 

То есть в данном случае человек будет главным компонентом, а нервная система - зависимым, подчиненным, и их создание и жизненный цикл будет происходить совместно, поэтому здесь лучше выбрать композицию.

---

**==Наследование==**. Является базовым принципом ООП и позволяет одному классу наследнику унаследовать функционал другого класса родительского. Нередко отношения наследования еще называют генерализацией или обобщением.

**==Реализация==**. Предполагает определение интерфейса и его реализация в классах. Например, имеется интерфейс `IMovable`, который реализуется в классе `Car`.

**==Ассоциация==**. Представляет собой тип отношения, при котором объекты одного типа неким образом связаны с объектами другого типа. Объекты будут ссылаться друг на друга. При этом они остаются полностью независимыми друг от друга.

**==Агрегация==**. Представляет тип отношений когда объект одного типа является частью объекта другого типа. Агрегация образует слабую связь между объектами. Все зависимые классы инициализируются вне основного объекта.

**==Композиция==**. Представляет собой тип отношений при котором один объект может принадлежать только другому объекту и никому другому. При композиции образуется сильная связь между объектами. При таком типе отношений основной объект полностью обеспечивает жизненный цикл объектов от которых он зависит.

---

Один из принципов проектирования гласит, что при создании системы классов надо программировать на уровне интерфейсов, а не их конкретных реализаций. Под интерфейсами в данном случае понимаются определение функционала без его конкретной реализации. То есть под определение попадают как интерфейсы, так и абстрактные классы, которые могут иметь абстрактные методы без реализации.

Часто приходится выбирать между интерфейсом и абстрактным классом. Если классы относятся к единой системе классификации, то выбирается абстрактный класс.

Как правило, абстрактные классы фокусируются на общем состоянии классов-наследников. В то время как интерфейсы строятся вокруг какого-либо общего действия.

Когда следует использовать абстрактные классы: Если надо определить общий функционал для родственных объектов, Если мы проектируем довольно большую функциональную единицу, Если нужно, чтобы все производные классы на всех уровнях наследования имели некоторую общую реализацию набора методов .

Когда следует использовать интерфейсы: Если мы проектируем небольшой функциональный тип, Если нам надо определить функционал для группы неких разрозненных объектов, которые могут быть никак не связаны между собой.