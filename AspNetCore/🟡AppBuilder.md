Для создания объекта класса `WebApplication` необходим  `WebApplicationBuilder`. Он создается через статический метод `CreateBuilder` приложения. В качестве параметра
в метод передаются аргументы командной строки приложения через параметр `args`.

Класс `WebApplicationBuilder` объединяет в себе конфигурацию хоста `IHostBuilder` для настройки самого приложения и хоста веб сервера `IWebHostBuilder`.  Он упрощает процесс настройки приложения, позволяя сконцентрироваться на компонентном подходе настройки.

```c# 
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();
app.Run();
```

---

**==Configuration==**. Представляет собой свойство с общим набором настроек приложения. С его помощью можно добавлять пользовательские провайдеры конфигурации, переопределять параметры или считывать необходимую информацию во время выполнения программы.

**==Services==**.  Свойство `builder.Services` представляет `IServiceCollection`, через которую регистрируются все зависимости приложения: от пользовательских репозиториев до встроенных компонентов `MVC`, `RazorPages` или `SignalR`. Регистрация выполняется стандартными методами `AddSingleton`, `AddScoped` и `AddTransient`. Все сервисы в последующем будут доступны в приложении через внедрение зависимостей `DI`.

**==Logging==.** Свойство `builder.Logging`, отвечающий за настройку источников и уровней логирования. В минимальном хосте по умолчанию подключены консоль и отладочный провайдеры. Для более сложных сценариев можно настроить уровень логирования или подключить сторонние фреймворки например, `Serilog` или `NLog` через расширения.

**==Environment==**. Свойство `builder.Environment` предоставляет `IHostEnvironment`, где доступно имя текущего окружения, путь к корню содержимого и к веб-корню. По умолчанию окружение определяется через переменную `ASPNETCORE_ENVIRONMENT`.

**==Host==**. Отвечает за глобальные параметры работы приложения хоста. Тут задаются источники конфигурации хоста: порядок и приоритет источников конфигурации настройки, политика создания и настройки контейнера внедрения зависимостей, валидация сервисов, выбор фабрики провайдера, а также параметры жизненного цикла хоста: как приложение слушает сигналы остановки и управляет фоновыми службами, очередями задач и планировщиками.

**==WebHost==**. Фокусируется на веб аспектах хоста. Тут задаются параметры: какой `HTTP` сервер используется и с какими ограничениями, на каких адресах и портах он принимает трафик,
где располагаются контентная и статическая части приложения, каким образом настроено шифрование  и проксирование, а также глобальные фильтры и `middleware` конвейер.