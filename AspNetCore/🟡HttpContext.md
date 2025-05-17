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

Сессия представляет механизм хранения данных на стороне сервера между запросами одного пользователя. Клиент получает уникальный идентификатор сессии `SessionID` и хранит его в виде `cookie`, и при каждом запросе отправляет его обратно. На основе него сервер восстанавливает набор пар ключ значение, ассоциированный с пользователем.

Для использование сессий нужно сначала подключить какие-нибудь реализации сервисов распределенного хранилища, затем подключить сервисы сессий через метод `AddSession`, затем зарегистрировать `middleware` в конвейере между `UseRouting` и `UseEndpoints`.

По умолчанию `AspNetCore` хранит данные в памяти серверного процесса `MemoryCache`.
Но при желании его можно сменить. Часто используют распределённое хранилище:

```C#
// MemoryCache
services.AddDistributedMemoryCache();

// Redis
services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
    options.InstanceName = "MyApp:";
});

// SqlServer
services.AddDistributedSqlServerCache(options =>
{
    options.ConnectionString = "...";
    options.SchemaName = "dbo";
    options.TableName   = "Sessions";
});
```

```c#
services.AddSession(options =>
{
    options.Cookie.Name = ".MyApp.Session";      // имя cookie
    options.Cookie.HttpOnly = true;              // запрещаем доступ из JS
    options.Cookie.IsEssential = true;           // обязательно даже при GDPR-откл.
    options.IdleTimeout = TimeSpan.FromMinutes(20); // время жизни сессии
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always; // по HTTPS
});
```

```c#
app.UseRouting();
app.UseSession();
app.UseEndpoints(ep =>{});
```

**==WebSockets==**. Представляет встроенный менеджер `WebSocket` соединений. С его помощью можно проверить, пришёл ли запрос на `WebSocket` соединение, и при также выполнить перевод текущего `HTTP` соединения в двунаправленный канал обмена сообщениями.

```c#
if (context.WebSockets.IsWebSocketRequest)
{
    var webSocket = await context.WebSockets.AcceptWebSocketAsync();
}
```

**==Response==**. Через свойство `Response`, представленное типом `HtttpResponse`, 
позволяет управлять ответом, который будет отправлен обратно клиенту.

```c#
app.Run(async (context) =>
{
    await context.Response.WriteAsync(request.PathBase);
});
```

**==Request==**. Свойство `Request` возвращает объект `HttpRequest`, в котором содержатся все данные входящего запроса: поток тела, заголовки, куки, параметры запроса и так далее.

```c#
app.Run(async (context) =>
{
    var request = context.Request;
    await context.Response.WriteAsync(request.PathBase);
});
```

[[HttpRequest]]
[[HttpResponse]]
[[AspNetCore/🟡Base|🟡Base]]

