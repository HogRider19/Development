Атрибуты представляют инструменты, которые позволяют встраивать в сборку дополнительные метаданные. Атрибуты могут применяться как ко всему типу,
так и к отдельным его частям методу, свойству. Основу атрибутов составляет 
класс `System.Attribute`, от которого образованы все остальные атрибуты.

Получить атрибут можно через метода расширения `GetCustomAttributes`, вызываемого у типа. Передаваемый `bool` параметр, определяет нужно ли искать атрибуты у наследников.
Получение атрибутов свойств и методов производится также через метод расширения.

```c#
var classAttributes = typeof(MyClass).GetCustomAttributes(true);
var serializableAttr = typeof(MyClass).GetCustomAttribute<SerializableAttribute>();
```

```c#
var methodInfo = typeof(MyClass).GetMethod("OldMethod");
var obsoleteAttr = methodInfo.GetCustomAttribute<ObsoleteAttribute>();

var propertyInfo = typeof(Person).GetProperty("Name");
var requiredAttr = propertyInfo.GetCustomAttribute<RequiredAttribute>();
```

С помощью атрибута `AttributeUsage` можно ограничить типы, к которым будет применяться атрибут. Например, нужно, чтобы атрибут применялся только к классам:

```c#
[AttributeUsage(AttributeTargets.Class)]
class AgeValidationAttribute : Attribute {}

[AttributeUsage(AttributeTargets.Class | AttributeTargets.Struct)]
class AgeValidationAttribute : Attribute {}
```

При установки атрибутов, также можно использовать специальные целевые спецификаторы, которые указывают точную цель применения атрибута: `return`, `method`, `param`, `property`.

```c#
[return: NotNull]                  
public string Method(
	[param: NotNull] string input,
	[field: NotNull] out string result) {}

[method: Obsolete] 
public void OldMethod() { }

[assembly: AssemblyVersion("1.0.0.0")]
[module: CLSCompliant(true)]
```

[[СSharp/🟡Base|🟡Base]]