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

---

**==Host==**. Содержит `HostString` запроса, где хранится доменное имя и опцио­нально порт, 
к примеру `example.com:5001`. Данное свойство полезно при построении ссылок.


```c#
var hostString = context.Request.Host;        
var host = host.Host; var port = host.Port;
```

**==PathBase, Path==**

- `PathBase` — «префиксный» сегмент, на котором смонтировано приложение (например, при использовании виртуального каталога).
    
- `Path` — остальная часть пути после `PathBase`.
    

csharp

КопироватьРедактировать

`// при запросе GET https://localhost:5001/app/api/values // и если приложение смонтировано на "/app" var basePath = context.Request.PathBase;    // "/app" // а вот дальше var path = context.Request.Path;            // "/api/values"`

---

**==QueryString, Query==**

- `QueryString` — строка запроса целиком (например, `"?id=5&page=2"`).
    
- `Query` — разобранная коллекция параметров `IQueryCollection`, где можно получить значения по ключу.
    

csharp

КопироватьРедактировать

`var rawQs = context.Request.QueryString;    // QueryString var id = context.Request.Query["id"];       // StringValues`

---

**==Headers==**  
Коллекция `IHeaderDictionary` всех заголовков запроса. Позволяет читать (и даже модифицировать) заголовки, например для CORS, логирования или аутентификации.

csharp

КопироватьРедактировать

`var userAgent = context.Request.Headers["User-Agent"].ToString(); if (context.Request.Headers.ContainsKey("X-My-Header")) {     var val = context.Request.Headers["X-My-Header"]; }`

---

**==Cookies==**  
Предоставляет доступ к кукам клиента: `IRequestCookieCollection`. Для чтения значений используйте индексирование.

csharp

КопироватьРедактировать

`if (context.Request.Cookies.TryGetValue("SessionId", out var sessionId)) {     // работаем с sessionId }`

---

**==Body, ContentLength, ContentType==**

- `Body` — поток `Stream`, из которого можно читать «сырое» тело запроса.
    
- `ContentLength` — длина тела в байтах (nullable).
    
- `ContentType` — MIME-тип тела (например, `application/json`).
    

csharp

КопироватьРедактировать

`context.Request.EnableBuffering();           // разрешаем многократное чтение using var reader = new StreamReader(context.Request.Body, leaveOpen: true); var bodyText = await reader.ReadToEndAsync(); context.Request.Body.Position = 0;           // сбрасываем позицию`

---

**==HasFormContentType, Form, ReadFormAsync()==**

- `HasFormContentType` — булево, указывает, что `ContentType` указывает на форму (`application/x-www-form-urlencoded` или `multipart/form-data`).
    
- `Form` — сразу разобранная коллекция полей формы (не нужно вручную вызывать `ReadFormAsync`, если вы доверяете дефолтному поведению).
    
- `ReadFormAsync()` — асинхронно разбирает тело и возвращает `IFormCollection`.
    

csharp

КопироватьРедактировать

`if (context.Request.HasFormContentType) {     var form = await context.Request.ReadFormAsync();     var name = form["name"];                // StringValues }`

---

**==Services==**  
Через `HttpContext.RequestServices` можно получать зависимости, зарегистрированные в DI-контейнере, но уже привязанные к текущему запросу. Например, скоуп-сервисы, настроенные как `Scoped`.

csharp

КопироватьРедактировать

`var db = context.RequestServices.GetRequiredService<MyDbContext>();`

---

**==HttpContext==**  
Свойство `HttpContext` позволяет из `HttpRequest` вернуться к полному контексту, если нужно получить доступ к ответу, сессии, фичам и пр.

csharp

КопироватьРедактировать

`var session = context.Request.HttpContext.Session;`

---

**==WebSockets==**  
Через `HttpRequest.HttpContext.WebSockets` можно проверить, пытается ли клиент выполнить WebSocket-апгрейд, и, если да, принять соединение.

csharp

КопироватьРедактировать

`if (context.Request.HttpContext.WebSockets.IsWebSocketRequest) {     var webSocket = await context.Request.HttpContext.WebSockets.AcceptWebSocketAsync();     // далее обмен сообщениями }`

---

**==Read and Buffering Middleware==**  
По умолчанию тело `Request.Body` можно читать один раз. Если нужно многократное чтение или буферизация, используйте `context.Request.EnableBuffering()` в начале pipeline. Это вставит middleware, копирующий данные в `MemoryBufferThreshold` (по умолч. 30 кБ) или на диск.

csharp

КопироватьРедактировать

`app.Use(async (ctx, next) => {     ctx.Request.EnableBuffering();     await next(); });`

---

**==Пример полного сценария==**

csharp

КопироватьРедактировать

`app.Use(async (context, next) => {     // Логируем базовые данные     var id = Guid.NewGuid().ToString();     var method = context.Request.Method;     var path = context.Request.Path;     var qs = context.Request.QueryString;     Console.WriteLine($"[{id}] {method} {path}{qs}");          // Буферизуем тело для следующих middleware     context.Request.EnableBuffering();          await next();          // Доп. лог после обработки     var status = context.Response.StatusCode;     Console.WriteLine($"[{id}] Response: {status}"); });  app.Run(async context => {     // Чтение тела, если был POST/PUT     if (context.Request.Method == "POST")     {         using var reader = new StreamReader(context.Request.Body, leaveOpen: true);         var body = await reader.ReadToEndAsync();         context.Request.Body.Position = 0;         await context.Response.WriteAsync($"You posted: {body}");     }     else     {         await context.Response.WriteAsync("Hello from HttpRequest demo!");     } });`

---

Этот конспект показывает основные грани класса `HttpRequest` в ASP.NET Core: от чтения стартовой строки и заголовков до буферизации тела, работы с формами и WebSockets, а также интеграции с DI и middleware.
