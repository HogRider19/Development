В `AspNetCore` каждый `HTTP` ответ представлен объектом `HttpResponse`, доступ к которому можно получить через свойство`Response` экземпляра `HttpContext`. Он позволяет управлять всеми аспектами ответа: статусом, заголовками, телом, куки, типом содержимого и другое.

**==StatusCode==**. Целочисленное свойство, представляющее `HTTP` статус ответа к примеру 
`100`, `200`, `404`, `500`. По умолчанию будет отправлен статус успешной обработки `200`.

```C#
context.Response.StatusCode = 404;
```

**==Headers==**. Коллекция `IHeaderDictionary`, содержащая все заголовки ответа. Можно использовать для установки `CORS` заголовков, управления кэшем браузера и прочего.

```c#
context.Response.Headers["Cache-Control"] = "no-cache"; context.Response.Headers.Add("X-Custom-Header", "value");
```

**==ContentType==**. Строка, определяющая `MIME` тип содержимого, например: `"text/html"`. 
Это поле необходимо для правильной интерпретации ответа браузером клиента.

```c#
context.Response.ContentType = "application/json";
```

**==ContentLength==**. Можно явно указать длину ответа в байтах. В большинстве случаев  данное поле поле выставляется системой автоматически на основе записанного тела запроса.

```c#
context.Response.ContentLength = responseBytes.Length;
```

**==Body==**. Поле, представленное типом потока данных `Stream`, в который пишется
тело ответа. Используется для ручной записи данных для отправки клиенту.

```c#
using var writer = new StreamWriter(context.Response.Body);
await writer.WriteAsync("Manual response body");
```

**==WriteAsync()==**. Метод для удобной записи текста в тело ответа. Устанавливает автоматически `ContentLength` и записывает переданный текст во внутренний поток для отправки клиенту.


```c#
await context.Response.WriteAsync("Hello, world!");
```

**==Redirect()==**. Метод для автоматического выполнения `HTTP` перенаправления
обычно `302`. Автоматически устанавливает поля `Location` и `StatusCode`.


```c#
context.Response.Redirect("/login");
```

**==Cookies==**. Свойство `Cookies` возвращает объект `IResponseCookies`, который позволяет добавлять или удалять куки в заголовках ответа, затем заголовки используются браузером.

```c#
context.Response.Cookies.Append("SessionId", "abc123",
	new CookieOptions { HttpOnly = true, Secure = true });  context.Response.Cookies.Delete("SessionId");
```

**==HasStarted==**. Флаг `bool`, показывающий, начала ли система уже отправлять
ответ клиенту. После начала передачи нельзя изменить заголовки и статус.

```c#
if (!context.Response.HasStarted) 
	{ context.Response.StatusCode = 404; }
```

Это может показаться странным, поскольку кажется логичным, что весь ответ должен формироваться только после завершения всех `middleware` в конвейере. Однако на
практике отправка может начаться  как только в `Response.Body` записываются данные.

Раннее начало отправки может произойти и по другим причинам. Например, если используется метод `Redirect()`или в конвейере работает `middleware`, которое пишет
тело ответа автоматически, например `UseStatusCodePages()` или `StaticFiles`.

Чтобы избежать проблем, любые заголовки, куки и статус-код следует задавать до первой записи в тело ответа. Если нужно гарантированно выполнить действия до отправки, можно воспользоваться `OnStarting()`, выполняется непосредственно перед отправкой заголовков.

```c#
context.Response.OnStarting(() =>
{
    context.Response.Headers["X-Before-Send"] = "true";
    return Task.CompletedTask;
});
```

**==OnStarting(), OnCompleted()==** Представляют методы регистрации колбэков. `OnStarting()` вызывается перед отправкой первого байта. `OnCompleted()` после завершения передачи.


```c#
app.Run(async (context) =>
{
    var clock = new Stopwatch();
    context.Response.OnStarting(() =>
	    { clock.Start(); return Task.CompletedTask; });
    
    context.Response.OnCompleted(() =>
	    { clock.Stop(); return Task.CompletedTask; });
    
    await context.Response.WriteAsync("Hello, world! ");
});
```

**==HttpContext==**. Позволяет из `HttpResponse` вернуться к полному контексту 
запроса, если нужен доступ к запросу или другим сервисам системы.

```c#
var requestPath = context.Response.HttpContext.Request.Path;
```

---

```c#
app.Use(async (context, next) => 
{    
	var id = Guid.NewGuid().ToString();
	Console.WriteLine($"[{id}] Handling {context.Request.Method}");
	
	context.Response.OnStarting(() => { 
		Console.WriteLine($"[{id}] Sending response ");
		return Task.CompletedTask;     
	});
		context.Response.OnCompleted(() => {  
		Console.WriteLine($"[{id}] Response sent."); 
		return Task.CompletedTask;     
	});
	
	await next();     
});
```