В `AspNetCore` имеется встроенная поддержка логгирования, что позволяет применять легирование с минимальными вкраплениями кода в функционал самого приложения.

Для логгирования данных нам необходим объект `ILogger<T>`. По умолчанию через механизм внедрения зависимостей уже предоставляется нам такой объект. Его можно получить как и любую другую зависимость в приложении. Также этот объект можно получить через свойство `Logger` объекта самого приложения `WebApplication`.

```c#
app.Logger.LogInformation($"Info")
```

При создании логгера для него указывается категория. Обычно в качестве категории логгера выступает класс, в котором используется логгер. В этом случае логгер типизируется классом.

```c#
app.Map("/hello", (ILogger<Program> logger) =>
    logger.LogInformation($"Hello, world!"));
```

```c#
app.Logger.LogCritical($"LogCritical");
app.Logger.LogInformation($"LogInformation");
app.Logger.LogWarning($"LogWarning");
app.Logger.LogError($"LogError");
```

В примерах в прошлой теме мы получали объект логгера, который добавляется через `DI`. Но мы можем также использовать фабрику логгера для его создания экземпляра логгера.

```c#
ILoggerFactory loggerFactory = LoggerFactory
	.Create(builder => builder.AddConsole());
	
ILogger logger = loggerFactory
	.CreateLogger<Program>();
	
app.Run(async (context) =>
{
    logger.LogInformation($"Requested Path: {context.Request.Path}");
    await context.Response.WriteAsync("Hello World!");
});
```

В данном случае с помощью метода `LoggerFactory.Create` создается фабрика логгера в виде объекта `ILoggerFactory`. В качестве параметра метод принимает делегат, который устанавливает некоторые настройки логгирования. В частности, метод `AddConsole()` объекта `ILoggingBuilder` устанавливает вывод сообщений всех догов на консоль.

В итоге мы получим тот же вывод сообщений на консоль. Но при этом преимущество использования фабрики логгеров состоит в том, что мы можем дополнительно настроить различные параметры логгирования, в частности, провайдер логгирования приложения.

---

Для глобальной установки настроек логгирования у объекта с типом `WebApplicationBuilder` определено свойство `Logging`, которое представляет настроечный тип `ILogingBuilder`. 

```c#
// 1. ClearProviders - удаляем все провайдеры логирования
builder.Logging.ClearProviders();

// 2. AddConfiguration - добавляем конфигурацию логирования из appsettings.json
builder.Logging.AddConfiguration(configuration.GetSection("Logging"));

// 3. SetMinimumLevel - устанавливаем минимальный уровень логирования
builder.Logging.SetMinimumLevel(LogLevel.Debug);

// 4. AddConsole - добавляем консольный логгер
builder.Logging.AddConsole();

// 5. AddJsonConsole - добавляем консольный логгер с JSON форматированием
builder.Logging.AddJsonConsole(options =>
{
    options.JsonWriterOptions = new System.Text.Json.JsonWriterOptions
    {
        Indented = true
    };
});

// 6. AddSimpleConsole - добавляем простое консольное форматирование
builder.Logging.AddSimpleConsole(options =>
{
    options.SingleLine = true;
    options.TimestampFormat = "HH:mm:ss ";
});

// 7. AddConsoleFormatter - добавляем кастомный форматтер для консоли
builder.Logging.AddConsoleFormatter<CustomConsoleFormatter, 
	ConsoleFormatterOptions>();

// 8. AddDebug - добавляем логгер отладки (вывод в Debug окно)
builder.Logging.AddDebug();

// 9. AddEventLog - добавляем логгер событий Windows (только для Windows)
if (OperatingSystem.IsWindows())
{
    builder.Logging.AddEventLog(eventLogSettings =>
    {
        eventLogSettings.SourceName = "MyApplication";
        eventLogSettings.LogName = "Application";
    });
}

// 10. AddFilter - добавляем фильтрацию для логгеров
builder.Logging.AddFilter("<namespace>", LogLevel.Warning);
builder.Logging.AddFilter((provider, category, logLevel) =>
{
    if (category.Contains("SpecialCategory") && logLevel >= LogLevel.Information)
    {
        return true;
    }
    return false;
});

// 11. AddProvider - добавляем кастомный провайдер логирования
builder.Logging.AddProvider(new CustomLoggerProvider());
```

---

Стандартная инфраструктура `AspNetCore` предоставляет, возможно, не самые удобные способы логгирования на консоль и в окне `Output`. Однако в то же время позволяет нам полностью определить свою логику ведения лога через создание своего провайдера логов.

```c#
public class FileLogger : ILogger, IDisposable
{
    string filePath;
    static object _lock = new object();
    
    public FileLogger(string path)
    {
        filePath = path;
    }
    
    public IDisposable BeginScope<TState>(TState state)
    {
        return this;
    }
    
    public void Dispose() { }
    
    public bool IsEnabled(LogLevel logLevel)
    {
        //return logLevel == LogLevel.Trace;
        return true;
    }
 
    public void Log<TState>(LogLevel logLevel, EventId eventId,
                TState state, Exception? exception,
                 Func<TState, Exception?, string> formatter)
    {
        lock (_lock)
        {
            File.AppendAllText(filePath, formatter(
	            state, exception) + Environment.NewLine);
        }
    }
}
```

Класс логгера должен реализовать интерфейс `ILogger`. Этот интерфейс определяет три метода: `BeginScope`: этот метод возвращает объект `IDisposable`, который представляет некоторую область видимости для логгера. `IsEnabled`: возвращает значения `bool`, которые указываtn, доступен ли логгер для использования. Здесь можно задать различную логику. В частности, в этот метод передается объект `LogLevel`, и мы можем, к примеру, задействовать логгер в зависимости от значения этого объекта. `Log`  этот метод предназначен для выполнения логгирования. В данном методе как раз и производится запись лога.

```C#
public class FileLoggerProvider : ILoggerProvider
{
    string path;
    
    public FileLoggerProvider(string path)
    {
        this.path = path;
    }
    
    public ILogger CreateLogger(string categoryName)
    {
        return new FileLogger(path);
    }
 
    public void Dispose() {}
}
```

Так же нам следует определить провайдер логгирования, который и будет создавать логгер. Он должен реализовать интерфейс `ILoggerProvider`. В этом интерфейсе определены два метода: `CreateLogger` создает и возвращает объект логгера, а также метод  `Dispose`.

```C#
public static class FileLoggerExtensions
{
    public static ILoggingBuilder AddFile(
	    this ILoggingBuilder builder, string filePath)
    {
        builder.AddProvider(new FileLoggerProvider(filePath));
        return builder;
    }
}
```

[[AspNetCore/🟡Base|🟡Base]]

