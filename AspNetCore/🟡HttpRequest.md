В `AspNetCore` каждый входящий `HTTP` запрос представлен объектом `HttpRequest`, доступ к которому можно получить через свойство `Request` экземпляра `HttpContext`. Это позволяет коду читать все детали запроса: метод, путь, заголовки, тело, куки, параметры и так далее.

**==Protocol==**. Возвращает строку с протоколом запроса, например `"HTTP/1.1"`. Полезно для логирования или выбора логики обработки в зависимость от версии протокола `HTTP`.

```c# 
var protocol = context.Request.Protocol;
```

**==Method==**. Содержит строку с методом запроса: `GET`, `POST`, `PUT`, `DELETE` и так далее. Часто используется в ветках контроллеров, чтобы различать логику в зависимости от метода.

```c#
var isPost = context.Request.Method.Equals("POST",
       StringComparison.OrdinalIgnoreCase);
```

**==Scheme==**. Возвращает строку схему `URI`: `"http"` или `"https"`. Можно использовать,
чтобы формировать абсолютные `URL` адреса в ответах или перенаправлениях.

```c# 
var isSecure = context.Request.Scheme == "https";
```

**==Host==**. Содержит `HostString` запроса, где хранится доменное имя и опцио­нально порт, 
к примеру `example.com:5001`. Данное свойство полезно при построении ссылок.


```c#
var hostString = context.Request.Host;        
var host = host.Host; var port = host.Port;
```

**==PathBase, Path==**. `PathBase` представляет префиксный сегмент, на котором смонтировано приложение. В `Path` записывается остальная часть пути после базового пути `PathBase`.

```c#
// при запросе GET https://localhost:5001/app/api/values 
// и если приложение смонтировано на "/app" 
var basePath = context.Request.PathBase;    // "/app" 
var path = context.Request.Path;            // "/api/values"
```

**==QueryString, Query==**. `QueryString` представляет объект типа `QueryString` который является оболочкой над целой строкой запроса. `Query` в свою очередь представляет разобранную коллекцию параметров `IQueryCollection`, где можно получить значения по ключу.

```c#
var rawQs = context.Request.QueryString;    // QueryString
var id = context.Request.Query["id"];       // StringValues
```

**==Headers==**. Представляет коллекцию `IHeaderDictionary` всех заголовков запроса. Позволяет читать и записывать заголовки, например для `CORS`, логирования или аутентификации.

```c#
var userAgent = context.Request.Headers["User-Agent"].ToString();
if (context.Request.Headers.ContainsKey("X-My-Header")) 
	var val = context.Request.Headers["X-My-Header"];
```

**==Cookies==**. Свойство возвращает коллекцию типа `IRequestCookieCollection`,
которая содержит все куки, отправленные клиентом в заголовке `Cookie`.

```c#
if (context.Request.Cookies.TryGetValue("SessionId", out var sessionId))
	Console.WriteLine(sessionId);
```

**==Body, ContentLength, ContentType==**. `Body` представляет поток `Stream`, из которого можно получить тело запроса. `ContentLength` длина тела. `ContentType` строка с `MIME` типом.

По умолчанию тело `Request.Body` можно читать один раз. Если нужно многократное чтение или буферизация, используйте `context.Request.EnableBuffering()` в начале `pipeline`. Это вставит `middleware`, копирующий данные запроса в `MemoryBufferThreshold` или на диск.

```c#
context.Request.EnableBuffering();
var reader = new StreamReader(
	context.Request.Body, leaveOpen: true);
var bodyText = await reader.ReadToEndAsync();
context.Request.Body.Position = 0;
```


**==HasFormContentType, Form, ReadFormAsync()==**. Поле `HasFormContentType` представляет флаг,  указывающий что `ContentType` указывает форму, то есть `multipart/form-data`. Поле `Form` сразу разобранная коллекция полей формы. Метод `ReadFormAsync()`  асинхронно разбирает тело и возвращает `IFormCollection`, которое позволяет получить поля полученный формы.

```c#
if (context.Request.HasFormContentType) 
{     
	var form = await context.Request.ReadFormAsync();
	var name = form["name"];
}
```


**==HttpContext==**. Свойство `HttpContext` позволяет из `HttpRequest` вернуться к полному контексту, если нужно получить доступ к ответу, сессии, фичам и прочим функциям.

```c#
var session = context.Request.HttpContext.Session;
```

---

```c#
app.Use(async (context, next) => {    
	// Логируем базовые данные 
	var id = Guid.NewGuid().ToString(); 
	var method = context.Request.Method; 
	var path = context.Request.Path; 
	var qs = context.Request.QueryString;
	Console.WriteLine($"[{id}] {method} {path}{qs}"); 
	// Буферизуем тело для следующих middleware 
	context.Request.EnableBuffering();  
	await next();       
	// Доп. лог после обработки  
	var status = context.Response.StatusCode; 
	Console.WriteLine($"[{id}] Response: {status}");
});

app.Run(async context => {  
	// Чтение тела, если был POST/PUT  
	if (context.Request.Method == "POST") 
	{        
		using var reader = new StreamReader(
			context.Request.Body, leaveOpen: true);
		var body = await reader.ReadToEndAsync();
		
		context.Request.Body.Position = 0; 
		await context.Response.WriteAsync($"You posted: {body}");
	}    
	else  
	{       
		await context.Response.WriteAsync("Hello from HttpRequest demo!"); 
	} 
});
```

