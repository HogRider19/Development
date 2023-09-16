**IWebHostEnvironment** - этот интерфейс, который предлагает ряд свойств, с помощью которых мы можем получить информацию об окружении

**app.Environment** - получение объекта IWebHostEnvironment **context.HostingEnvironment** - получение объекта IWebHostEnvironment

**Env.IsEnvironment(string envName)** - true, если совпадает EnvName **Env.IsDevelopment()** - возвращает true, если имя среды - Development **Env.IsStaging()** - возвращает true, если имя среды - Staging **Env.IsProduction()** - возвращает true, если имя среды - Production

## IWebHostEnvironment свойства:

1. **ApplicationName** - хранит имя приложения
2. **EnvironmentName** - название среды, в которой запущено приложение
3. **ContentRootPath** - хранит путь к корневой папке приложения
4. **WebRootPath** - хранит путь к папке, в которой хранится статический контент приложения. По умолчанию это папка wwwroot
5. **ContentRootFileProvider** - возвращает реализацию интерфейса Microsoft.AspNetCore.FileProviders.IFileProvider, которая может использоваться для чтения файлов из папки ContentRootPath
6. **WebRootFileProvider** - возвращает реализацию интерфейса Microsoft.AspNetCore.FileProviders.IFileProvider, которая может использоваться для чтения файлов из папки WebRootPath