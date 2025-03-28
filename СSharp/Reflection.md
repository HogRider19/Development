Рефлексия представляет собой процесс, который позволяет программам инспектировать и изменять свою структуру и поведение во время выполнения.
Функционал рефлексии содержится в пространстве имен `System.Reflection`. 

В C# есть блок под названием Assembly, который автоматически создается компилятором после успешной компиляции кода. Assembly состоит из двух частей: промежуточного языка и метаданных, которые не компилируются в машинный код.

![[Pasted image 20231106011817.png]]

Класс Type представляет изучаемый тип, инкапсулируя всю информацию о нем. С помощью его свойств и методов можно получить эту информацию.

**Атрибуты(System.Attribute)** - представляют специальные инструменты, которые позволяют встраивать в сборку дополнительные метаданные. 

При создание собственного атрибуты следует создать класс и унаследовать его от абстрактного класса Attribute. Далее с помощью конструкторов сохранить необходимую информацию. Во время исполнения кода можно будет достать данные из атрибута по средствам рефлексии и настроить определенное поведение.

С помощью атрибутов можно указывать среде выполнения и компилятору определенное поведение. Одним из таких атрибутов является `MethodImplAttribute`. Он принимает объект перечисления `MethodImplOptions`, который может иметь значения:

**AggressiveInlining** - если это возможно, во всех местах применять встраивание метода.
**Synchronized** - делает метод потокобезопасным за счет применения к нему монитора.
**NoOptimization** - указывает компилятору не оптимизировать работы данного метода.
**NoInlining** - указывает компилятору, что данный метод не должен встраиваться.

---
Основные классы рефлексии, представленные в пространстве имен `System.Reflection`, включают в себя набор классов для обращения к различным атрибутам сущностей:

1. **Assembly** - класс, представляющий сборку.
2. **AssemblyName** - класс, хранящий информацию о сборке.
3. **MemberInfo** - базовый абстрактный класc для классов info.
4. **EventInfo** -класс, хранящий информацию о событии.
5. **FieldInfo** - хранит информацию об определенном поле типа.
6. **MethodInfo** - хранит информацию об определенном методе.
7. **PropertyInfo** - хранит информацию о свойстве.
8. **ConstructorInfo** - класс, представляющий конструктор.
9. **Module** - класс, позволяющий получить доступ к модулю сборки.
10. **ParameterInfo** - класс, хранящий информацию о параметре метода.

Основным классом рефлексии считается класс `Type`, который представляет функционал для изучения различных типов данных на их внутреннюю структуру:

1. **FindMembers()** - возвращает массив объектов MemberInfo
2. **GetConstructors()** - возвращает все конструкторы данного типа
3. **GetEvents()** - возвращает все события данного типа
4. **GetFields()** - возвращает все поля данного типа
5. **GetInterfaces()** - получает все реализуемые интерфейсы
6. **GetMembers()** - возвращает все члены типа в виде массива
7. **GetMethods()** - получает все методы типа в виде массива
8. **GetProperties()** - получает все свойства в виде массива
9. **Name** - возвращает имя типа
10. **Assembly** - возвращает название сборки, где определен тип
11. **Namespace** - возвращает название пространства имен, где определен тип

[[СSharp/🟡Base|🟡Base]]