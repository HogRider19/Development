Приложение в `AspNetCore` представлено классом `WebApplication`. Данный класс отвечает за запуск и остановку приложения, за настройку обработчиков маршрутизации `middleware`,
а также отвечает за регистрации конечных точек обработки конечных запросов приложения.

```c#
WebApplicationBuilder builder = WebApplication.CreateBuilder();
WebApplication app = builder.Build();

await app.StartAsync();
await app.StopAsync();
app.Start();
app.Stop();
```

**==Configuration==**. Свойство `app.Configuration` возвращает `IConfiguration`, собранную на этапе `CreateBuilder`. Позволяет считывать настройки из ранее созданных провайдеров.

**==Environment==**. Свойство `app.Environment` предоставляет доступ к имени окружения `EnvironmentName`, корневому каталогу `ContentRootPath` и статики `WebRootPath`.

**==Services==**. Свойство `app.Services` представляет сервисный провайдер, через который
можно получать зарегистрированные ранее в билдере приложения зависимости.

**==Logger==**  Свойство `app.Logger` представленное обобщенным типом `ILogger<Program>` позволяет получить доступ к логгеру с уже настроенным провайдерами логирования. 

**==Lifetime==**. Свойство `app.Lifetime` возвращает интерфейс `IHostApplicationLifetime`,
через которое можно подписаться на события жизненного цикла приложения.

**==Urls==**. Свойство `app.Urls` коллекция строк, прослушиваемых веб сервером,
данные настройки задаются на уровне конфигурации приложения веб хоста. 