  Ошибки в приложении можно условно разделить на два типа: исключения, которые возникают в процессе выполнения кода, и стандартные ошибки протокола `HTTP`.
Обычные исключения могут быть полезны для разработчика в процессе создания приложения, но простые пользователи не должны будут их видеть в браузере.

**==UseDeveloperExceptionPage==**. Этот `middleware` предназначен для обработки исключений в приложении, которое находится в процессе разработки. Однако вручную нам его не надо добавлять, поскольку оно добавляется автоматически. Так, если мы посмотрим на код
класса `WebApplicationBuilder`, то мы там можем увидеть там следующие строки:

```c#
if (context.HostingEnvironment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
```

Если приложение находится в состоянии разработки, то с помощью этого `middleware`
приложение перехватывает исключения и выводит информацию о них разработчику.
Информация будет показываться отдельной страницей в браузере пользователя.

**==UseExceptionHandler==**. Это не самая лучшая ситуация, и нередко все-таки возникает необходимость дать пользователям некоторую информацию о том, что же все-таки произошло. Либо потребуется как-то обработать данную ситуацию. Для этих целей
можно использовать еще один встроенный компонент `ExceptionHandlerMiddleware`, который подключается с помощью метода `UseExceptionHandler()`. Он перенаправляет
при возникновении исключения на некоторый адрес и позволяет обработать исключение.

```c#
app.Environment.EnvironmentName = "Production";
 
// если приложение не находится в процессе разработки
// перенаправляем по адресу "/error"
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
}
 
// middleware, которое обрабатывает исключение
app.Map("/error", app => app.Run(async context =>
{
    context.Response.StatusCode = 500;
    await context.Response.WriteAsync("Error 500");
}));
```

Однако данный способ имеет недостаток мы можем напрямую обратиться по адресу
`/error` и получить тот же ответ от сервера. Но в этом случае мы можем использовать другую версию метода `UseExceptionHandler()`, которая принимает `IApplicationBuilder`.

```c#
app.Environment.EnvironmentName = "Production";
 
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler(builcer => 
    {
	    builce.Run(async context =>
	    {
	        context.Response.StatusCode = 500;
	        await context.Response.WriteAsync("Error 500");
	    });
    });
}
 
app.Run(async (context) =>
{
    int a = 5;
    int b = 0;
    int c = a / b;
    await context.Response.WriteAsync($"c = {c}");
});
 
app.Run();
```

---

В отличие от исключений стандартный функционал проекта `AspNetCore` почти никак не обрабатывает ошибки `HTTP`, например, в случае если ресурс не найден. При обращении к несуществующему ресурсу мы увидим в браузере пустую страницу, и только через консоль веб-браузера мы сможем увидеть статусный код. Либо браузер отобразит свою страницу.

Но с помощью компонента `StatusCodePagesMiddleware` можно добавить в проект отправку информации о статусном коде. Для этого применяется метод `app.UseStatusCodePages()`:

```c#
var builder = WebApplication.CreateBuilder();
var app = builder.Build();
 
// обработка ошибок HTTP
app.UseStatusCodePages();
 
app.Map("/hello", () => "Hello ASP.NET Core");
 
app.Run();
```

Метод `UseStatusCodePages()` следует вызывать ближе к началу конвейера обработки запроса, до добавления для работы со статическими файлами и  конечных точек.

Сообщение, отправляемое методом `UseStatusCodePages()` по умолчанию, не очень информативное. Однако одна из версий метода позволяет настроить отправляемое пользователю сообщение. В частности, мы можем изменить вызов метода так:

```c#
var builder = WebApplication.CreateBuilder();
var app = builder.Build();
 
// обработка ошибок HTTP
app.UseStatusCodePages("text/plain",
	"Error: Resource Not Found. Status code: {0}");
 
app.Map("/hello", () => "Hello ASP.NET Core");
 
app.Run();
```

Еще одна версия метода `UseStatusCodePages()` позволяет детально задать обработчик ошибок. Она принимает делегат, параметр которого - объект `StatusCodeContext`. В свою очередь, объект `StatusCodeContext` имеет свойство` HttpContext`, из которого мы можем получить всю информацию о запросе и настроить отправку ответа пользователю. Например:

```c#
app.UseStatusCodePages(async statusCodeContext =>
{
    var response = statusCodeContext.HttpContext.Response;
    var path = statusCodeContext.HttpContext.Request.Path;
    
    response.ContentType = "text/plain; charset=UTF-8";
    if (response.StatusCode == 403)
    {
        await response.WriteAsync($"Path: {path}. Access Denied ");
    }
    else if (response.StatusCode == 404)
    {
        await response.WriteAsync($"Resource {path} Not Found");
    }
});
```

Вместо метода `UseStatusCodePages()` мы также можем использовать еще пару других, которые обрабатывают ошибки `HTTP`. С помощью `UseStatusCodePagesWithRedirects()`
можно выполнить переадресацию на определенный обрабатывающий код метод.

```c#
app.UseStatusCodePagesWithRedirects("/error/{0}");
 
app.Map("/hello", () => "Hello ASP.NET Core");
app.Map("/error/{statusCode}", (int statusCode) => $"Error: {statusCode}");
```

Здесь будет идти перенаправление по адресу `/error/{0}`. В качестве параметра через плейсхолдер `{0}` будет передаваться статусный код ошибки. Но теперь при обращении к несуществующему ресурсу клиент получит статусный код `302 / Found`. То есть формально несуществующий ресурс будет существовать, просто статусный код `302` будет указывать, что ресурс перемещен на другое место по пути `/error/404`. Подобное поведение может быть неудобно, особенно с точки зрения поисковой индексации, и в этом случае мы можем применить другой компонентный метод `app.UseStatusCodePagesWithReExecute()`:

```c#
app.UseStatusCodePagesWithReExecute("/error/{0}");
 
app.Map("/hello", () => "Hello ASP.NET Core");
app.Map("/error/{statusCode}", (int statusCode) => $"Error: {statusCode}");
 
app.Run();
```

В качестве параметра метод `UseStatusCodePagesWithReExecute()` приминает путь к ресурсу, который будет обрабатывать ошибку. То есть в данном случае при возникновении ошибки будет вызываться конечная точка. Формально мы получим тот же ответ, так как так же будет идти перенаправление на путь `/error/404`. Но теперь браузер получит статусный код `404`.