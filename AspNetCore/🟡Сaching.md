Кэширование представляет собой механизм сохранение данных в отельном месте для более быстрого доступа к ним в будущем. Применение кэширование может значительно повысить производительность приложения, уменьшая количество обращений к источникам данных.

==**PreCaching**==. Путем анализа разработчик определяет наиболее часто запрашиваемые или данные, которые могут снизить производительность приложения. Эти данные кэшируются при старте приложения. Затем после запуска, приложение берет эти данные из кэша вместо запрашивания из внешнего источника. Минус этого подхода необходимость синхронизации с внешним источником данных, особенно когда работают разные команды разработчиков.

**==OnDemandCaching==**. Когда данные необходимы, приложение сначала обращается в кэш, если кэше найдены соответствующие данные, то они используются это называется `СacheHit`. Если данные в кэше отсутствуют это называется `CacheMiss`, то приложение извлекает из базы данных и кэшируют для последующих запросов. Минус стратегии необходимость делать запрос к базе данных, который снижает производительность итогового приложения.

---

Самым простым способом кэширования в `AspNetCore` представляет использование объекта `Memory.IMemoryCache`, который позволяет сохранять данные в кэше на сервере. Применяя методы интерфейса `IMemoryCache`, мы можем управлять системой кэша в приложении. 

```c#
public interface IMemoryCache : IDisposable
{
	// Добавляет в кэш или перезаписывает запись с key.
	ICacheEntry CreateEntry(object key);
	
	// добавляет в кэш элемент, применяя опции MemoryCacheEntryOptions
	object Set(object key, object value, MemoryCacheEntryOptions options)
	
	// Пытается получить элемент по ключу key.
	bool TryGetValue(object key, out object? value);
	
	// Удаляет из кэша элемент по ключу key
	void Remove(object key);
}
```

Платформа `AspNetCore` предоставляет встроенную реализацию интерфейса `IMemoryCache` 
в виде `MemoryCache`, который используется как реализация по умолчанию для сервиса `IMemoryCache` и который инкапсулирует все объекты кэша в виде словаря `Dictionary`.

Чтобы получить сервис `IMemoryCache` в приложении его необходимо добавить в коллекцию сервисов с помощью `AddMemoryCache()`. По сути этот сервис устанавливает зависимость для интерфейса `IMemoryCache`, сопоставляя его через синглтон c объектом типа `MemoryCache`.


```c#
builder.Services.AddMemoryCache();

app.Map("/", (IMemoryCache cache) =>  
{  
	cache.Set("Key", "Value", new MemoryCacheEntryOptions()
		.SetAbsoluteExpiration(TimeSpan.FromMinutes(5)));
});
```

Для установки параметров кэширования в метод `Set()` в качестве третьего параметра передается объект `MemoryCacheEntryOptions`, который устанавливает ряд настроек.

```c#
var cacheOptions = new MemoryCacheEntryOptions
{
    // Абсолютное время истечения (конкретная дата/время)
    AbsoluteExpiration = DateTimeOffset.Now.AddHours(2),
    
    // Альтернатива: абсолютное время истечения относительно текущего момента
    AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1),
    
    // Скользящее время истечения (сбрасывается при каждом обращении)
    SlidingExpiration = TimeSpan.FromMinutes(15),
    
    // Приоритет кэширования
    Priority = CacheItemPriority.High,
    
    // Размер элемента в кэше (для управления кэшем на основе размера)
    Size = 1024,
    
    // Токены изменения (для принудительного удаления при изменениях)
    ExpirationTokens = {
        new CancellationChangeToken(
	        new CancellationTokenSource(TimeSpan.FromMinutes(30)).Token)
    },
    
    // Коллбэки при удалении из кэша
    PostEvictionCallbacks = {
        new PostEvictionCallbackRegistration
        {
            EvictionCallback = (key, value, reason, state) => 
                Console.WriteLine($"Элемент {key} удален. Причина: {reason}"),
            State = null
        }
    }
};

// Использование опций при добавлении в кэш
_memoryCache.Set("cache_key", "cache_value", cacheOptions);
```

---

Одну из форм форм кэширования представляет распределенное кэширование. Представляет механизм общего кеша у нескольких серверов приложений. Распределенный кэш обладает такими преимуществами, как согласованность данных между запросами нескольких разных серверов, независимость от приложения, запуска, перезапуска и развертывания, отсутствия влияния на локальную память. Соответственно применение распределенного кэша может повысить производительность и масштабируемость серверного приложения `AspNetCore`.

В `C#` распределенный кэш представлен специальным интерфейсом `IDistributedCache`.
Для управления кэшем этот тип предоставляет ряд методов. Сам интерфейс выглядит так:

```c#
public interface IDistributedCache
{
	byte[]? Get(string key);
	void Set(string key, byte[] value, DistributedCacheEntryOptions options);
	
	Task<byte[]?> GetAsync(string key, CancellationToken token);
	Task SetAsync(string key,byte[] value, DistributedCacheEntryOptions options,
	  CancellationToken token);
	  
	void Remove(string key);
	Task RemoveAsync(string key, CancellationToken token);
	
	void Refresh(string key);
	Task RefreshAsync(string key, CancellationToken token));
}
```

Из перечисления методов видно, что мы можем сохранить в кэш данные либо в виде массива байт `byte[]`, либо в виде строки `string`. И обратно получить данные из кэша также можно только в виде объектов `byte[]` и `string`. Соответственно если наши данные представляют другой тип, то нам понадобиться механизм сериализации и десериализации.

```c#
// Добавление Redis
builder.Services.AddStackExchangeRedisCache(options => {
    options.Configuration = "localhost";
    options.InstanceName = "local";
});
```

---

Компрессия или сжатие ответа представляет один из инструментов, которые позволяют уменьшить объем отправляемых клиенту данных без потери состояния самих данных.

Для компрессии ответа в рамках `AspNetCore` мы можем использовать либо внутренние средства конкретного веб-сервера: `DynamicCompressionModule` в `IIS`, модуль `mod_deflate` в `Apache` или инструменты компрессии в `Nginx`. Однако эти инструменты не всегда бывают доступны, особенно когда приложение напрямую хостится в рамках таких веб серверов как `HTTP.sys` или `Kestrel`. Тогда мы можем использовать компонентом `ResponseCompression`. 

```c#
var builder = WebApplication.CreateBuilder(args);
 
builder.Services.AddResponseCompression(
	options => options.EnableForHttps = true);
 
var app = builder.Build();
 
app.UseResponseCompression();
app.MapGet("/", () => "Hello, world!");
app.Run();
```

Следует учитывать, что для протокола `HTTPS` по умолчанию отключено сжатие, поэтому в вызове данного метода подключается сжатие и для `HTTPS`. На уровне протокола `HTTP` это выражается также в установлении заголовка `content-encoding`, который сервер вместе с ответом посылает клиенту. По умолчанию этот заголовок имеет значение `br`, который говорит о том, что используется сжатие в специальный формат сжатия данных `Brotli`.

При использования компрессии при отправке ответа нам обязательно надо установить 
`MIME` тип отправляемых данных. Заголовок типа ответа устанавливался автоматически при использовании `ResultsApi`,  `MVC` или `RazorPages`. Однако не все `middleware` в конвейере `AspNetCore` автоматически устанавливают заголовки. Например, при обработке запроса с помощью метода `Run` заголовки не устанавливаются, и их надо устанавливать вручную.

```c#
builder.Services.AddResponseCompression(
	options => options.EnableForHttps = true);
 
var app = builder.Build();

app.UseResponseCompression();
app.Run(async (context) =>
{
    context.Response.ContentType = "text/plain";
    await context.Response.WriteAsync("Hello, world!");
});
```

---

Кроме кэширования результатов действий контроллера для оптимизации работы приложения можно применять кэширование файлов статического контента.

Для кэширования статических файлов при их отправке следует установить соответствующие заголовки. Для этого `StaticFilesMiddleware` передается объект `StaticFileOptions`, у которого устанавливается свойство `OnPrepareResponse`.

Для кэширования статических файлов следует установить соответствующие заголовки.
Для этого `StaticFilesMiddleware` передается объект `StaticFileOptions`, у которого устанавливается свойство `OnPrepareResponse`, имеющее доступ к `Http` контексту:

```c#
app.UseStaticFiles(new StaticFileOptions()
{
    OnPrepareResponse = ctx =>
    {
	    ctx.Context.Response.Headers.Add("Cache-Control", "public,max-age=600");
	}
});
```

Свойство `OnPrepareResponse` представляет делегат, который в качестве параметра принимает объект `StaticFileResponseContext`. Через этот объект можно установить заголовки ответа, который будут посылаться вместе со статическими файлами.

В данном случае устанавливается заголовок `Cache-Control`, который  задает параметры кэширования. Значение `public` указывает на кеширование браузера и прокси серверов.
У кэширования статических файлов есть недостаток: если мы изменим содержимое файла,
то при повторном обращении браузер продолжит извлекать устаревшие файлы из кэша.

Для решения этой проблемы можно добавлять к статическим файлам версионность
и при каждом изменении файла сервером соответственно менять версию файла:

```html
<link rel="stylesheet" href="/css/site.css?v=123" />
<script src="app.js?v=123"></script>
```

---

Платформа `AspNetCore` позволяет кэшировать ответ приложения, и для этого применяет сервис `IOutputCacheStore` и специальный компонент конвейера `OutputCacheMiddleware`. 

Для кэширования ответа приложения нам надо прежде всего настроить приложения.
Для этого следует выполнить два шага. Прежде всего, добавить в коллекцию сервисов приложения все необходимые сервисы кеша с помощью метода `AddOutputCache()`.
Также нужно добить в конвейер `OutputCacheMiddleware` через `UseOutputCache()`.

```c#
var builder = WebApplication.CreateBuilder(args);
 
builder.Services.AddOutputCache();
var app = builder.Build();
 
app.UseOutputCache();
app.MapGet("/", () => "Hello, world!").CacheOutput();
app.MapGet("/", [CacheOutput] () => "Hello, world!");

app.Run();
```

[[AspNetCore/🟡Base|🟡Base]]
