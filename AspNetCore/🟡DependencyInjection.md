Внедрение зависимостей `DependencyInjection`  представляет процесс предоставления внешней зависимости программному компоненту. Является специфичной формой инверсии управления. Такие объекты связаны между собой через абстракции, например, через интерфейсы, что делает всю систему более гибкой и также более адаптируемой.
 
Для уменьшения связности объектов системы, сами объекты заменяются на некоторые абстрактные зависимости. Тем не менее остается проблема управления подобными зависимостями, особенно если это касается больших программных приложений.

Внедрение зависимостей является одной из реализаций концепции `InversionOfControl`.
Инверсия управления подразумевает, что поток выполнения программы управляется не прикладным кодом, а кодом фреймворка. В обычной программе программист сам решает,
в какой последовательности делать вызовы процедур. Но если используется фреймворк, программист может разместить свой код в определенных точках выполнения, затем запустить главную функцию фреймворка, которая обеспечит всё выполнение.

Нередко для установки зависимостей в таких системах используются специальные контейнеры `IoC (Inversion of Control)`. Такие контейнеры служат своего рода фабриками, которые устанавливают связь между абстракциями и конкретными
объектами и также как правило, управляют процессом созданием этих объектов.

[[[🟡InversionOfControl]]]

Преимуществом `AspNetCore` в этом отношении является то, что фреймворк уже по умолчанию имеет встроенный контейнер внедрения зависимостей, который представлен интерфейсом `IServiceProvider`. Cами зависимости еще называются сервисами, собственно поэтому контейнер можно назвать провайдером сервисов. Этот контейнер отвечает за сопоставление зависимостей с конкретными типами и за внедрение зависимостей.

```c#
WebApplicationBuilder builder = WebApplication.CreateBuilder();
IServiceCollection allServices = builder.Services;  // IServiceProvider
```

Cервис в коллекции `IServiceCollection` представляет объект `ServiceDescriptor`,
который содержит информацию об интерфейсе, типе и жизненном цикле сервиса.

```C#
foreach (var svc in builder.Services)
{
	var type = svc.ServiceType;
	var impType = svc.ImplementationType;
	var lifeTime = svc.Lifetime;
}
```

---

Кроме ряда подключаемых по умолчанию сервисов имеется еще ряд встроенных
сервисов, которые мы можем подключать в приложение при необходимости.

Все по умолчанию представленные сервисы, регистрируются в приложение с помощью методов расширений интерфейса `IServiceCollection`, имеющих форму `Add[Name]()`.

```c#
var builder = WebApplication.CreateBuilder();
builder.Services.AddMvc();
```

После добавления сервиса его можно получить в любой части приложения. Для
получения сервиса могут применяться различные способы в зависимости от ситуации.
Например можно использовать свойство `Services` объекта `WebApplication`, которое предоставляет провайдер сервисов объект `IServiceProvider`. Через внедрение в конструктор класса сервиса или метод `Invoke` у компонента или конструктор. Через свойство `RequestServices` у контекста запроса. Через класс `WebApplicationBuilder`.

Для получения сервиса у провайдера сервиса вызывается обобщенный метод
`GetService()` или `GetRequiredService()`, которые типизируются типом сервиса.

```c#
var some1 = app.Services.GetService<SomeService1>();
var some2 = app.Services.GetRequiredService<SomeService2>();
```

Паттерн получения сервисов через `IServiceProvider` еще называется `Servicelocator`
и как правило, рекомендуется использовать другие способы, но тем не менее в рамках `AspNetCore` в принципе мы можем использовать подобную функциональность особенно когда требуется внедрить сервис с более коротким жизненным циклом в другой сервис.

---

Система позволяет управлять жизненным циклом внедряемых в приложении сервисов.
С точки зрения жизненного цикла сервисы могут представлять один из типов:

**==Transient==**. Жизненный цикл при котором, при каждом обращении к сервису создается новый объект сервиса. В течение одного запроса может быть несколько обращений к сервису, соответственно при каждом обращении будет создаваться новый экземпляр сервиса. 

```c#
builder.Services.AddTransient<ICounter, RandomCounter>();
```

**==Scoped==**. Для каждого запроса создается свой объект сервиса. Если в течение одного запроса есть несколько обращений к сервису, то будет использоваться один и тот же объект сервиса.

```c#
builder.Services.AddScoped<ICounter, RandomCounter>();
```

**==Singleton==**. Такой объект сервиса создается при первом обращении к нему, все 
последующие запросы используют один и тот же ранее созданный объект сервиса.

```c#
builder.Services.AddSingleton<ICounter, RandomCounter>();
```

Внедрять сервисы через механизм `DI` можно только с увеличенным или таким же жизненным циклом, внедрение в обратном порядке возможно только через сервисный локатор, то есть с использованием внедрение `IServiceProvider` и вызов `CreateScope()`. 

```c#
builder.Services.AddSingleton<ISingletonWithScoped, SingletonWithScoped>();
builder.Services.AddScoped<IScopedService, ScopedService>();

public class SingletonWithScoped : ISingletonWithScoped
{
    private readonly IServiceProvider _serviceProvider;
    
    public SingletonWithScoped(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public void DoWork()
    {
        using (var scope = _serviceProvider.CreateScope())
        {
            var scopedService = scope.ServiceProvider
	            .GetRequiredService<IScopedService>();
            scopedService.PerformAction();
        }
    }
}
```

---

По умолчанию при внедрении зависимостей в `AspNetCore` одна зависимость сопоставляется с одним типом. Однако бывают ситуации, когда требуется отойти от этой привязки один к одному. Первая ситуация: для одной зависимости необходимо зарегистрировать сразу несколько реализаций. А также когда для нескольких зависимостей нужен один объект.

При регистрации под одной абстракцией нескольких реализаций сервиса, 
коллекцию реализаций можно получить через внедрение `IEnumerable<IType>`.

```c#
builder.Services.AddTransient<IHelloService, RuHelloService>();
builder.Services.AddTransient<IHelloService, EnHelloService>();

////
public HelloMiddleware(RequestDelegate _, IEnumerable<IHelloService> hs)
{
	this.helloServices = helloServices;
}
```

В обратной ситуации, когда для нескольких абстракций зарегистрирована одна реализация, получение сервиса происходит через стандартный механизм внедрения интерфейса. Но стоит учитывать, что при регистрации данных сервисов как `Singleton`, для каждой абстракции будет создан свой объект реализации. Данную проблему можно обойти 
передав уже созданный объект как параметр в метод `AddSingleton()`. 

```c#
builder.Services.AddSingleton<IGenerator, ValueStorage>();  // Два Singleton
builder.Services.AddSingleton<IReader, ValueStorage>();
```

```c#
builder.Services.AddSingleton<ValueStorage>();              // Один Singleton
builder.Services.AddSingleton<IGenerator>(serv =>  
			serv.GetRequiredService<ValueStorage>());
builder.Services.AddSingleton<IReader>(serv => 
			serv.GetRequiredService<ValueStorage>());
```

[[AspNetCore/🟡Base|🟡Base]]

