Для взаимодействия с окружением, в котором запущено приложение, предоставлен интерфейс IWebHostEnvironment. Этот интерфейс предлагает ряд свойств, с помощью которых мы можем получить информацию об окружении:

**ApplicationName** - хранит имя условное имя приложения.
**EnvironmentName** - название окружения, в которой запущено приложение.
**ContentRootPath** - хранит путь к корневой папке приложения.
**WebRootPath** - хранит путь к папке, в которой хранится статический контент.

При разработке мы можем использовать эти свойства. Но наиболее часто при разработке придется сталкиваться со свойством EnvironmentName.

По умолчанию имеются три варианта значений для этого свойства: Development, Staging и Production. Свойство задается через установку переменной среды `ASPNETCORE_ENVIRONMENT`. Текущее значение задается в файле launchSettings.json.

**app.Environment** - получение объекта IWebHostEnvironment 
**context.HostingEnvironment** - получение объекта IWebHostEnvironment

**Env.IsEnvironment(string envName)** - true, если совпадает EnvName
**Env.IsDevelopment()** - возвращает true, если имя среды - Development 
**Env.IsStaging()** - возвращает true, если имя среды - Staging 
**Env.IsProduction()** - возвращает true, если имя среды - Production
