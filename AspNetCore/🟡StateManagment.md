Приложение `AspNetCore` может хранить состояние. Это могут быть как какие-то глобальные данные, так и данные, которые непосредственно относятся к запросу и пользователю. 
И в зависимости от вида данных, существуют различные способы для их хранения.

**==HttpContext.Items==**. Один из простейших способов хранения данных представляет объект типа `IDictionary<object, object>`. Эта коллекция предназначена для таких данных, связанных с текущим запросом. После завершения запроса данные из нее удаляются. 

Эта коллекция может применяться например, если у нас обработка запроса вовлекает множество `middleware`, и мы хотим, чтобы для этих компонентов были общие данные.

```c#
app.Use(async (context, next) =>
{
    context.Items["text"] = "Hello from HttpContext.Items";
    await next.Invoke();
});
 
app.Run(async (context) => 
{
	await context.Response.WriteAsync($"Text: {context.Items["text"]}")
});
```

**==Cookies==**. Куки представляют самый простой способ сохранить данные пользователя. Куки хранятся на компьютере пользователя и могут устанавливаться как на сервере, так и на клиенте.  Для оптимизации системы максимальный размер ограничен `4096` байтами.

Для работы с куками можно использовать контекст запроса `HttpContext`. Чтобы получить куки, которые приходят вместе с запросом, надо использовать коллекцию `Request.Cookies` объекта `HttpContext`. Эта коллекция представляет объект `IRequestCookieCollection`, в котором каждый элемент представлен объектом типа `KeyValuePair<string, string>`.

```c#
app.Use(async (context, next) => 
{
	if (context.Request.Cookies.ContainsKey("myCookies"))
	    string name = context.Request.Cookies["myCookies"];
	    
	if (context.Request.Cookies.TryGetValue("myCookies", out string cookies))
	{
		await context.Response.WriteAsync($"Text: {cookies");
	}
	else
	{
		await context.Response.WriteAsync($"Hello, world!");
	}
});
```

Для установки `Cookies`, которые отправляются в ответ клиенту, применяется объект `context.Response.Cookies`, который представляет интерфейс `IResponseCookies`. Этот интерфейс определяет два метода: `Append(string key, string value)` и `Delete(key)`.

```c#
app.Run(async (context) =>
{
    if (context.Request.Cookies.ContainsKey("name"))
    {
        string? name = context.Request.Cookies["name"];
        await context.Response.WriteAsync($"Hello {name}!");
    }
    else
    {
        context.Response.Cookies.Append("name", "Tom");
        await context.Response.WriteAsync("Hello World!");
    }
});
```

[[HTTP]]

**==Sessions==**. Сессия в `AspNetCore` это механизм хранения временных данных, связанных с конкретным пользователем между `HTTP` запросами. Сессия идентифицируется уникальным идентификатором, обычно через `cookie`, и позволяет сохранять данные на стороне сервера в течение ограниченного времени, пока пользователь взаимодействует с приложением. Она особенно полезна для хранения информации, не требующей постоянного сохранения, например, содержимого корзины, временных настроек пользователя и так далее.

Для хранения состояния сессии на сервере создается словарь, который хранится в кэше и который существует для всех запросов из одного браузера в течение некоторого времени. На клиенте хранится идентификатор сессии в куках. Этот идентификатор посылается на сервер с каждым запросом. Сервер использует идентификатор для извлечения нужных данных из сессии. Эти куки удаляются только при завершении сессии. Но если сервер получает куки истекшей сессии, то для этих `cookie` сервер создаст новую сессию.

Сервер хранит данные сессии в течение ограниченного времени после последнего запроса. По умолчанию этот промежуток равен `20` минутам, но его также можно сконфигурировать.
Следует учитывать, что данные сессии специфичны для одного браузера и не разделяются между ними. То есть для каждого браузера компьютера будет создан свой набор данных.

Все сессии работают поверх объекта `IDistributedCache`, и `AspNetCore` предоставляет встроенную реализацию `IDistributedCache`, которую мы можем использовать.

```c#
var builder = WebApplication.CreateBuilder();

// добавляем IDistributedMemoryCache
builder.Services.AddDistributedMemoryCache();
// добавляем сервисы сессии
builder.Services.AddSession();
 
var app = builder.Build();

// добавляем middleware для работы с сессиями
app.UseSession();
 
app.Run(async (context) =>
{
    if (context.Session.Keys.Contains("name"))
        await context.Response.WriteAsync(
	        $"{context.Session.GetString("name")}!");
    else
    {
        context.Session.SetString("name", "Tom");
        await context.Response.WriteAsync("Hello World!");
    }
});
 
app.Run();
```

Если мы вдруг не используем `app.UseSession()` или попробуем обратиться к сессии
до применения этого метода, то получим исключение `InvalidOperationException`.

После вызова метода `app.UseSession()` мы сможем управлять сессиями через свойство `HttpContext.Session`, которое представляет специальный объект интерфейса `ISession`. 
Идентификатор сессии будет храниться в специальном `cookie` `.AspNetCore.Session`.

Также по умолчанию куки имеют настройку `CookieHttpOnly=true`, поэтому они не доступны для клиентских скриптов из браузера. Но мы можем переопределить ряд настроек сессии с помощью свойств объекта `SessionOptions`, который передается как параметр в делегат, принимаемый одной из перегрузок метода, добавляющего компонент `UseSession()`.

Зачастую сессии хранят простые строки. Если надо сохранить какой-то сложный объект, то его надо сериализовать, а при получении обратно десериализовать. Как правило, для этого определяются методы расширения для объекта `ISession` с нужным нам функционалом.

```c#
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession();
var session = context.Session;
```

[[AspNetCore/🟡Base|🟡Base]]
