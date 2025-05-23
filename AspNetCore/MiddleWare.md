Основная задача приложения это обработка входящих запросов. Обработка запроса в `AspNetCore` устроена по принципу конвейера цепочки обязанностей, который состоит
из компонентов. Подобные компоненты конвейера цепочки называются `middleware`.

При получении запроса сначала данные запроса получает первый компонент в конвейере. После обработки запроса компонент `middleware` он может закончить обработку запроса такой компонент еще называется терминальным компонентом . Либо он может передать данные запроса для обработки далее по конвейеру. После обработки запроса последним компонентом, данные запроса возвращаются к предыдущему компоненту обратно.

![[Pasted image 20231209160752.png]]

Компоненты middleware встраиваются с помощью методов расширений `Run`, `Map` и `Use`
интерфейса `IApplicationBuilder`. Класс `WebApplication` реализует этот интерфейс и поэтому позволяет добавлять компоненты `middleware` с помощью данных методов.

Каждый компонент `middleware` может быть определен как метод встроенный 
`inline` компонент, либо может быть вынесен в отдельно объявленный класс.

Компоненты создаются один раз и существуют в течение всего жизненного цикла приложения. То есть для обработки запросов используются одни и те же компоненты.

При получении запроса сервер формирует на его основе объект `HttpContext`, который содержит всю необходимую информацию о запросе. Эта информация посредством объекта `HttpContext` передается всем компонентам middleware в приложении.

**Task RequestDelegate(HttpContext context, next)** - делегат определения middleware.

Для создания компонентов middleware используется делегат `RequestDelegate`, который выполняет некоторое действие и принимает контекст запроса - объект `HttpContext`. 

Стоит отметить, что ASP.NET Core уже по умолчанию предоставляет ряд встроенных компонентов middleware, которые применяются для часто встречающихся задач:

**Authentication** - предоставляет поддержку аутентификации.
**Authorization** - предоставляет поддержку авторизации.
**Cookie Policy** - отслеживает согласие пользователя на хранение информации.
**CORS** - обеспечивает поддержку кроссдоменных запросов политики CORS.
**DeveloperExceptionPage** - генерирует веб-страницу с информацией об ошибке.
**Diagnostics** - набор middleware, для диагностики при разработке приложения.
**HealthCheck** - проверяет работоспособность приложения на определенном url.
**HTTPSRedirection** - перенаправляет все запросы HTTP на HTTPS протоколов.

**MVC** - обеспечивает функционал фреймворка MVC.
**RequestLocalization** - обеспечивает поддержку локализации.
**ResponseCaching** - позволяет кэшировать результаты запросов.
**ResponseCompression** - обеспечивает сжатие ответа клиенту.
**URLRewrite** - предоставляет функциональность URL Rewriting.
**EndpointRouting** - предоставляет механизм маршрутизации.
**Session** - предоставляет поддержку сессий.
**StaticFiles** - предоставляет поддержку обработки статических файлов.
**WebSockets** - добавляет поддержку протокола WebSockets.

Для встраивания этих компонентов в конвейер обработки запроса для интерфейса `IApplicationBuilder` определены методы расширения типа `UseXXX`.

---

У интерфейса `IApplicationBuilder` есть набор методов расширений, для встраивания компонентов middleware, которые уже к собранному приложению. 

**==Run(ReqDel)==**. Добавляет терминальный компонент, который завершает обработку запроса. Поэтому соответственно он не вызывает никакие другие компоненты и обработку запроса дальше - следующим в конвейере компонентам не передает.

Прежде всего, не стоит путать метод Run(), который определен в классе WebApplication и который запускает приложение, и метод расширения Run(), который встраивает компонент middleware. Это два разных метода, которые выполняют разные задачи.

```c#
app.Run(async (context) => await context.Response.WriteAsync("Hello, world"));
```

**==Use(RedDel)==** - добавляет компонент middleware, который позволяет передать обработку запроса далее следующим в конвейере компонентам через `next.Invoke()`.

Первый параметр делегата `Func`, который передается в метод `Use()`, представляет объект `HttpContext`. Этот объект позволяет получить данные запроса.

Второй параметр делегата представляет другой делегат - `Func<Task>` или `RequestDelegate`. Этот делегат представляет следующий в конвейере компонент middleware, которому будет передаваться обработка запроса.

При использовании метода `Use` и передаче выполнения следующему делегату следует учитывать, что не рекомендуется вызывать метод `next.Invoke` после записи ответа. 

Компонент middleware должен либо генерировать ответ, либо вызывать следующий делегат посредством `next.Invoke`, но не выполнять оба этих действия одновременно. Так как согласно документации последующие изменения объекта `Response` могут привести к нарушению протокола, например, будет послано больше байт, чем указано.

Обработчик необязательно должен вызывать следующий в конвейере компонент. Вместо этого он может завершить обработку запроса. В этом случае он может выступать в роли такого же терминального компонента, как и метод `Run`.

```c#
app.Use(async (context, next) => await next.Invoke(context));
```

**==Map(path, appBuilder=>)==** - применяется для создания ветки конвейера, которая будет обрабатывать запрос по определенному пути. Этот метод реализован как метод расширения для типа `IApplicationBuilder` и имеет ряд перегруженных версий.

В качестве параметра `path` метод принимает путь запроса, с которым будет сопоставляться ветка. А второй параметр представляет делегат, в который передается объект `IApplicationBuilder` и в котором будет создаваться ветка конвейера.

Ветка конвейера, которая создается в методе `Map()`, может иметь вложенные ветки, которые обрабатывают подзапросы. При этом путь для дочерних веток конвейера будет обрабатываться относительно пути родительской ветки. 

```c# 
app.Map("/path", appBuilder => ...);
```

**==UseWhen(cond, appBuilder=>)==** - создает middleware, которое на основании некоторого условия позволяет создать ответвление конвейера при обработке запроса.

В качестве параметра он принимает делегат `Func<HttpContext, bool>` - некоторое условие, которому должен соответствовать запрос. В этот делегат передается объект `HttpContext`. Если запрос соответствует условию, то делегат должен вернуть `true`.

Второй параметр делегат `Action<IApplicationBuilder>` представляет некоторые действия над` IApplicationBuilder`, который передается в делегат параметром.

Стоит отметить, что создание ветки происходит один раз при запуске приложения. То есть код в делегате второго параметра будет выполнен лишь один раз при старте.

```c#
app.UseWhen(context => cond, appBuilder => {});
```

**==MapWhen(context=>bool, buider=>)==** - аналогичен `UseWhen`, но в отличии от него не может передавать контекст в основную ветку конвейера выполнения .

Основным отличием этих методов расширения является то, что если метод `UseWhen` не содержит в себе терминального компонента, то в конце выполнения он передаст контекст выполнения в основную ветку, в то время как `MapWhen` завершит обработку.

```c#
app.UseWhen(context => cond, appBuilder = ...);
```

---

В прошлых случаях используемые компоненты middleware фактически представляют из себя методы, то есть так называемые `inline middleware`. Однако также система позволяет определять компоненты middleware в виде отдельных классов.

Класс компонента должен иметь конструктор, который принимает параметр типа `RequestDelegate`. Через этот параметр можно получить  делегат запроса, который  будет находиться следующим в конвейере обработки запроса.

Также в классе должен быть определен метод, который должен называться `Invoke` или `InvokeAsync`. Причем этот метод должен возвращать объект `Task` и принимать в качестве параметра контекст запроса. Данный метод и будет обрабатывать запрос.

**app.UseMiddleware\<MType>()** - регистрирует компонент MType в приложении.

```c#
public class NameMiddleware
{
    private readonly RequestDelegate next;
  
    public TokenMiddleware(RequestDelegate next)
    {
        this.next = next;
    }
  
    public async Task InvokeAsync(HttpContext context)
    {
		await next.Invoke(context);
    }
}
```

[[AspNetCore/🟡Base|🟡Base]]
