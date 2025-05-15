В `AspNetCore` для каждого запроса создаётся экземпляр класса `HttpContext`.
Он инкапсулирует всю информацию о входящем запросе и позволяет управлять формированием ответа. Контекст передаётся по цепочке `middleware` в приложении
через делегат `RequestDelegate`. Также экземпляр контекста можно получить через `DI`.

**==Connection==**. Возвращает объект `HttpConnectionInfo`, который инкапсулирует детали соединения: `IP` адрес клиента, порты, информацию о безопасности `TLS`. Это бывает полезно, например, чтобы логировать отправителя или реализовать `IP` фильтрацию.

```c#
var isSecure = context.Connection.ClientCertificate != null;
var remoteIp = context.Connection.RemoteIpAddress;    // IPAddress
var localIp = context.Connection.LocalIpAddress;      // IPAddress
var remotePort = context.Connection.RemotePort;       // int
var localPort  = context.Connection.LocalPort;        // int
```

**==Features==**. Cодержит низкоуровневые интерфейсы для расширения возможностей `HTTP` обработки. Через неё можно получить, скажем, поддержку `WebSockets` или `HTTP/2 Push`. Каждый `middleware` имеет возможность добавить или прочитать элемент из этой коллекции.

Механизм может выглядеть так: `middleware` принимает контекст, на основе его параметров создает некоторый экземпляр класса, затем устанавливает этот экземпляр в коллекцию `Features` через метод `Set<T>(T? instance)`, затем другие `middleware` могут уже при необходимости получить экземпляр этого класса через метод `Features.Get<T>()`.

```c#
var webSocketFeature    = context.Features.Get<IHttpWebSocketFeature>(); 
var bodyControl         = context.Features.Get<IHttpBodyControlFeature>();
var authFeature         = context.Features.Get<IHttpAuthenticationFeature>();
var lifetime            = context.Features.Get<IHttpRequestLifetimeFeature>();
var sendFile            = context.Features.Get<IHttpSendFileFeature>();
var maxSizeFeature      = context.Features.Get<IHttpMaxRequestBodySizeFeature>();
var upgradeFeature      = context.Features.Get<IHttpUpgradeFeature>();
var tlsFeature          = context.Features.Get<ITlsHandshakeFeature>();
```

**==Items==**. Свойство `Items` предоставляет словарь `IDictionary<object,object?>` для обмена данными между разными уровнями `middleware` в рамках одного запроса. Это удобно, когда нужно, например, засечь время начала обработки и передать его дальше для логирования.


```c#
context.Items["data"] = 100;
var data = (int)context.Items["data"];
```

**==RequestServices==**. Через `RequestServices` доступен локальный провайдер зависимостей `IServiceProvider`, который позволяет каждому `middleware` или компоненту получить зарегистрированные сервисы. Это центральный механизм внедрения зависимостей.


```c#
var service1 = context.RequestServices.GetService<SomeType1>();
var service2 = context.RequestServices.GetRequiredService<SomeType2>();
```

**==TraceIdentifier==**. Свойство `TraceIdentifier` выдаёт уникальный для каждого запроса `GUID`, который удобно использовать в логах и трассировке. А с помощью `User`  можно работать с информацией об аутентифицированном пользователе: проверять роли, извлекать клаймы.

```c#
var guid = context.TraceIdentifier;
```

**==Session==**. Свойство `Session` предоставляет доступ к объекту `ISession`, в котором можно хранить пользовательские данные между запросами. Перед тем как работать с сессиями, необходимо подключить `middleware` вызовом `app.UseSession()` и зарегистрировать хранилище в памяти или `Redis` через метод `services.AddDistributedMemoryCache()`. 

```c#
context.Session.SetString("CartId", cartId);
var cartId = context.Session.GetString("CartId");
```

**==WebSockets==**. 

---

**==HttpRequest (Request)==**  
Свойство `Request` возвращает объект `HttpRequest`, в котором содержатся все подробности входящего запроса: поток тела (`Body`, `BodyReader`), заголовки (`Headers`), куки (`Cookies`), параметры строки запроса (`Query`, `QueryString`), маршрутные данные (`RouteValues`), метод (`Method`), путь (`Path`, `PathBase`), схема (`Scheme`) и признак HTTPS (`IsHttps`). Например, чтобы получить данные формы из POST-запроса, можно вызвать:

csharp

КопироватьРедактировать

`var form = await context.Request.ReadFormAsync(); var username = form["username"];`

В том же объекте можно проверить длину и тип содержимого (`ContentLength`, `ContentType`), что полезно для валидации или ограничения загрузок.

---

**==HttpResponse (Response)==**  
Через свойство `Response` вы управляете всем, что будет отправлено клиенту. Объект `HttpResponse` предоставляет:

- **Body** и **BodyWriter**: поток и PipeWriter для записи в тело ответа.
    
- **HasStarted**: флаг, показывающий, началась ли уже отправка, чтобы избежать конфликтов.
    
- Заголовки (`Headers`), включая удобный словарь `IHeaderDictionary` с методами `Append` и т. п.
    
- Куки (`Cookies`), `StatusCode`, `ContentType`, `ContentLength`, а также ссылку обратно на `HttpContext`.
    

Чтобы установить заголовок, достаточно:

csharp

КопироватьРедактировать

`context.Response.Headers.Append("X-Custom", "Value"); context.Response.ContentType = "application/json";`

---

**Метод Redirect**  
Выполняет перенаправление клиента на другой URL. По умолчанию это временный (302) редирект, но флаг `permanent: true` делает его постоянным (301).

csharp

КопироватьРедактировать

`// временно context.Response.Redirect("/signin"); // постоянно context.Response.Redirect("/new-location", permanent: true);`

**Метод WriteAsJsonAsync**  
Сериализует объект в JSON и устанавливает заголовок `Content-Type: application/json`. Это самый простой способ вернуть данные API без лишнего кода.

csharp

КопироватьРедактировать

`var dto = new { Id = 42, Name = "Gadget" }; await context.Response.WriteAsJsonAsync(dto);`

**Метод WriteAsync**  
Записывает текстовую строку в тело ответа. Подходит, когда нужен простой ответ без сериализации.

csharp

КопироватьРедактировать

`await context.Response.WriteAsync("Привет из middleware!");`

**Метод SendFileAsync**  
Отправляет файл напрямую из файловой системы в ответ, минуя буферизацию. Кроме строки с путём существует перегрузка для `IFileInfo`. Так удобно отдавать большие статические файлы.

csharp

КопироватьРедактировать

`await context.Response.SendFileAsync("wwwroot/files/manual.pdf");`

---

Таким образом, `HttpContext` создаётся единоразово на каждый запрос, передаётся по middleware-цепочке и служит универсальным контейнером для работы с HTTP-входом и выходом, хранения временных данных и доступа к инфраструктуре приложения.


## Основные свойства `HttpContext`

- **Connection** (`HttpConnectionInfo`)  
    Содержит информацию о клиентском соединении: IP-адрес, порт, протокол TLS.
    
    csharp
    
    КопироватьРедактировать
    
    `var remoteIp = context.Connection.RemoteIpAddress;`
    
- **Features** (`IFeatureCollection`)  
    Коллекция низкоуровневых интерфейсов (features), реализующих различные аспекты HTTP-обработки.
    
    csharp
    
    КопироватьРедактировать
    
    `var hasWebSockets = context.Features.Get<IHttpWebSocketFeature>() != null;`
    
- **Items** (`IDictionary<object, object?>`)  
    Словарь для обмена данными между middleware на протяжении одного запроса.
    
    csharp
    
    КопироватьРедактировать
    
    `context.Items["StartTime"] = DateTime.UtcNow;`
    
- **RequestServices** (`IServiceProvider`)  
    Локальный провайдер сервисов, позволяет получать scoped- и transient-зависимости.
    
    csharp
    
    КопироватьРедактировать
    
    `var logger = context.RequestServices.GetService<ILogger<Startup>>();`
    
- **TraceIdentifier** (`string`)  
    Уникальный идентификатор запроса для трассировки и логирования.
    
    csharp
    
    КопироватьРедактировать
    
    `logger.LogInformation("Request {TraceId}", context.TraceIdentifier);`
    
- **User** (`ClaimsPrincipal`)  
    Принципал текущего аутентифицированного пользователя.
    
    csharp
    
    КопироватьРедактировать
    
    `if (context.User.Identity.IsAuthenticated) { … }`
    
- **WebSockets** (`WebSocketManager`)  
    Позволяет обрабатывать WebSocket-запросы и отправлять сообщения.
    
    csharp
    
    КопироватьРедактировать
    
    `if (context.WebSockets.IsWebSocketRequest) {     var ws = await context.WebSockets.AcceptWebSocketAsync();     // … }`
    
- **Session** (`ISession`)  
    Хранит данные сессии (при включённой поддержке), доступно после вызова `app.UseSession()`.
    
    csharp
    
    КопироватьРедактировать
    
    `context.Session.SetString("CartId", cartId);`
    
- **Request** и **Response**  
    Два ключевых свойства для доступа к объектам `HttpRequest` и `HttpResponse` (описаны ниже).
    

---

## Работа с **HttpResponse** (`context.Response`)

Свойство `Response` возвращает объект `HttpResponse`, позволяющий гибко формировать ответ:

### `Redirect(string location, bool permanent = false)`

Выполняет HTTP-переадресацию на указанный URL.

- **Пример** (временный редирект, статус 302):
    
    csharp
    
    КопироватьРедактировать
    
    `context.Response.Redirect("/home/index");`
    
- **Пример** (постоянный, 301):
    
    csharp
    
    КопироватьРедактировать
    
    `context.Response.Redirect("/new-url", permanent: true);`
    

---

### `WriteAsJsonAsync<T>(T value, JsonSerializerOptions? options = null, CancellationToken token = default)`

Сериализует объект в JSON и устанавливает `Content-Type: application/json`. Требует пакет `Microsoft.AspNetCore.Http.Json`.

csharp

КопироватьРедактировать

`var product = new { Id = 1, Name = "Notebook" }; await context.Response.WriteAsJsonAsync(product);`

---

### `WriteAsync(string text, CancellationToken cancellationToken = default)`

Записывает текст в тело ответа. Подходит для простых случаев, когда не нужна сериализация.

csharp

КопироватьРедактировать

`await context.Response.WriteAsync("Hello, world!");`

---

### `SendFileAsync(string path, long offset = 0, long? count = null, CancellationToken cancellationToken = default)`

Отправляет файл из файловой системы. Удобно для отдачи статического контента без буферизации в памяти.

csharp

КопироватьРедактировать

`await context.Response.SendFileAsync("wwwroot/files/report.pdf");`

Также есть перегрузка, принимающая `IFileInfo`.

---

## Работа с **HttpRequest** (`context.Request`)

Свойство `Request` возвращает объект `HttpRequest`, где сосредоточена информация о входящем запросе:

- **Чтение тела**
    
    - Поток: `context.Request.Body` (Stream)
        
    - PipeReader: `context.Request.BodyReader`
        
- **Чтение формы**
    
    - `IFormCollection Form` (синхронно, если уже прочитано)
        
    - Асинхронно:
        
        csharp
        
        КопироватьРедактировать
        
        `var form = await context.Request.ReadFormAsync(); var name = form["username"];`
        
- **Заголовки, куки и параметры**
    
    csharp
    
    КопироватьРедактировать
    
    `var accept = context.Request.Headers["Accept"]; var token = context.Request.Cookies["AuthToken"]; var q = context.Request.Query["search"];`
    
- **Метаданные маршрута**
    
    csharp
    
    КопироватьРедактировать
    
    `var id = context.Request.RouteValues["id"];`
    
- **Другие свойства**  
    `Method`, `Path`, `PathBase`, `QueryString`, `Scheme`, `IsHttps`.
    

---

Таким образом, `HttpContext` — это центральный объект, создаваемый на каждый запрос и позволяющий как читать всю информацию о запросе (через `Request`), так и формировать ответ (через `Response`), а также обмениваться данными между компонентами приложения.


---
---
---

Все данные запроса передаются в middleware через объект `HttpContext`. Этот объект инкапсулирует информацию о запросе, позволяет управлять ответом и так далее. 

**Microsoft.AspNetCore.Http.HttpContext** - объект инкапсулирующий запрос и ответ клиенту для передачи его по цепочке конвейеров в `MiddleWare`.

Для создания компонентов middleware используется делегат `RequestDelegate`, который выполняет некоторое действие и принимает контекст запроса - объект `HttpContext`. Контекст запроса инкапсулирует в себе информацию о забросе в виде свойств:

**Connection** - представляет информацию о подключении данного запроса.
**Features** - получает коллекцию HTTP функциональности текущего запроса.
**Items** - получает или устанавливает коллекцию пар для хранения некоторых данных.
**Request** - возвращает объект HttpRequest, который хранит информацию о запросе.
**RequestServices** - объект IServiceProvider для работы с зарегистрированными сервисами.
**Response** - возвращает объект HttpResponse, который позволяет управлять ответом.
**TraceIdentifier** - представляет идентификатор запроса для логов трассировки.
**WebSockets** - возвращает объект для управления подключениями WebSocket.
**User** - представляет пользователя, ассоциированного с этим запросом.
**Session** - хранит данные сессии для текущего запроса.

Для получения информации о запросе и управления ответом, который будет отправлен клиенту, контекст запроса имеет два свойства `Request` и `Response`.

---

**==Response==**. Свойство объекта `HttpContext`, представляет объект `HttpResponse` и задает, что будет отравляться в виде ответа. Для этого `HttpResponse` определяет свойства:

**Body** - получает или устанавливает тело ответа в виде объекта Stream.
**BodyWriter** - возвращает объект типа PipeWriter для записи ответа.
**HasStarted** - возвращает true, если отправка ответа уже началась.

**ContentLength** - получает или устанавливает заголовок Content-Length.
**ContentType** - получает или устанавливает заголовок Content-Type.
**StatusCode** - возвращает или устанавливает статусный код ответа.
**Host** - получает или устанавливает заголовок Host.

**Headers** - возвращает заголовки ответа.
**Cookies** - возвращает куки, отправляемые в ответе.
**HttpContext** - возвращает контекст, связанный с данным ответом.

Для установки заголовков применяется свойство `Headers`, которое представляет тип `IHeaderDictionary`. Для большинства стандартных заголовков в этом интерфейсе определены одноименные свойства. Другие можно добавить через метод `Append()`.

Чтобы отправить ответ, мы можем использовать ряд методов класса `HttpResponse`:

**Redirect()** - выполняет переадресацию (временную или постоянную) на другой ресурс.
**WriteAsJson()** - отправляет ответ в виде объектов в формате JSON.
**WriteAsync()** - отправляет некоторое содержимое в виде текста. 
**SendFileAsync()** - отправляет файл.

Для отправки файлов применяется метод `SendFileAsync()`, который получает либо путь к файлу в виде строки, либо информацию о файле в виде объекта `IFileInfo`.

Таким способом можно отправлять файлы с кодом разметки, или с постоянным для выдачи контентом, но стоит отметить, что в ASP.NET Core уже имеется встроенный middleware, который позволяет упростить работу со статическими файлами.

---

**==Request==**. Свойство `Request` объекта `HttpContext` представляет объект `HttpRequest` и хранит информацию о запросе, отправленного клиентом, в виде следующих свойств:

**Body** - предоставляет тело запроса в виде объекта Stream.
**BodyReader** - возвращает объект типа PipeReader для чтения тела запроса.

**ContentLength** - получает или устанавливает заголовок Content-Length.
**ContentType** - получает или устанавливает заголовок Content-Type.
**Cookies** - возвращает коллекцию куки (объект Cookies).
**Headers** - возвращает заголовки запроса.
**Host** - получает или устанавливает заголовок Host.

**Form** - получает или устанавливает тело запроса в виде форм.
**HasFormContentType** - проверяет наличие заголовка Content-Type
**HttpContext** - возвращает связанный с данным запросом HttpContext

**IsHttps** - возвращает true, если применяется протокол https.
**Method** - получает или устанавливает метод HTTP.
**Path** - путь запроса в виде объекта RequestPath.
**PathBase** - получает или устанавливает базовый путь запроса.
**Protocol** - получает или устанавливает протокол, например, HTTP.
**Query** - возвращает коллекцию параметров из строки запроса.
**QueryString** - получает или устанавливает строку запроса.
**RouteValues** - получает данные маршрута для текущего запроса.
**Scheme** - получает или устанавливает схему запроса HTTP.

Нередко данные отправляются на сервер с помощью форм, обычно в запросе типа POST. Для получения таких данных в классе `HttpRequest` определено свойство `Form`.

Свойство `Request.Form` возвращает объект `IFormCollection`, где по ключу можно получить значение элемента. В качестве ключей выступает названия полей формы.

[[AspNetCore/🟡Base|🟡Base]]

