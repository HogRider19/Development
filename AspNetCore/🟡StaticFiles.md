Статические фалы приложения можно отправлять вручную через `Response.SendFileAsync`. Однако `AspNetCore` также предоставляет специальный встроенный `middleware`, который подключается `UseStaticFiles()` и который упрощает работу со статическими файлами.

При использовании этого `middleware` применяются некоторые условности. В частности, по умолчанию для определения пути хранения статических файлов в проекте используются два параметра `ContentRoot` и `WebRoot`, а сама статика помещается в `ContentRoot/WebRoot`.

На стадии разработки параметр `ContentRoot` соответствует каталогу текущего проекта.
А параметр `WebRoot` по умолчанию представляет папку `wwwroot` в рамках корневого каталога. То есть, исходя из значений по умолчанию, статические файлы следует
располагать в папке `wwwroot`, которая должна находиться в текущем проекте.
Однако эти параметры при необходимости можно переопределить.

В разных типах проектов`AspNetCore` данная папка может уже быть по умолчанию в проекте, а может отсутствовать. Например, в проекте по типу `Empty` данная директория отсутствует.

```c#
var builder = WebApplication.CreateBuilder();
	new WebApplicationOptions { WebRootPath = "static"}
var app = builder.Build();
 
app.UseStaticFiles();
app.Run(async (context) => {});
app.Run();
```

После применения этого `middleware` сервер начнет выдавать статические файлы
по маршрутам, советующим пути к сатирическому контенту относительно `static`. 

Перегрузка метода `UseStaticFiles()` позволяет сопоставить пути с определенными каталогами. В примере ниже, первый вызов `app.UseStaticFiles()` обрабатывает запросы
к файлам в папке `wwwroot`. Второй вызов обрабатывает запросы в `/pages`, сопоставляя данные запросы с папкой `wwwroot/html`. К примеру, по запросу `x/pages/index.html`
мы можем обратиться к файлу, находящимся по пути `wwwroot/html/index.html`.

```C#
app.UseStaticFiles();
app.UseStaticFiles(new StaticFileOptions()
{
    FileProvider = new PhysicalFileProvider(
            Path.Combine(Directory.GetCurrentDirectory(), @"wwwroot\html")),
    RequestPath = new PathString("/pages")
});
```

---

С помощью специального метода расширения `UseDefaultFiles()` можно настроить отправку статических веб-страниц по умолчанию без обращения к ним по полному пути.

```c#
var app = builder.Build();
app.UseDefaultFiles();
app.UseStaticFiles();
```

По умолчанию данный `middleware` обслуживает страницы `default.html` и `index.html`. 
Но при желании их можно сменить через передачу `DefaultFilesOptions` в этот метод.

```c#
DefaultFilesOptions options = new DefaultFilesOptions();
options.DefaultFileNames.Clear();
options.DefaultFileNames.Add("hello.html");
app.UseDefaultFiles(options);
```

Также есть метод расширения с компонентом `UseDirectoryBrowser` позволяет пользователям просматривать содержимое каталогов на сайте в виде списка.

```c#
var builder = WebApplication.CreateBuilder();
var app = builder.Build();
 
app.UseDirectoryBrowser();
app.UseStaticFiles();
 
app.Run();
```

---

Метод расширения `UseFileServer()` объединяет функциональность сразу всех трех вышеописанных методов `UseStaticFiles`, `UseDefaultFiles` и `UseDirectoryBrowser`.

```c#
app.UseFileServer();
```

По умолчанию этот метод позволяет обрабатывать статические файлы и отправлять файлы по умолчанию типа `index.html`. Если нам надо еще включить просмотр каталогов, то мы можем использовать перегрузку данного метода, передав в него специальный флаг `true`.

```c#
app.UseFileServer(new FileServerOptions
{
    EnableDirectoryBrowsing = true,
    EnableDefaultFiles = false,
    RequestPath = new PathString("/pages"),
    FileProvider = new PhysicalFileProvider(
	    Path.Combine(Directory.GetCurrentDirectory(), @"wwwroot\html")),
});
```

---

В `AspNetCore 9.0 ` был добавлен новый компонент `MapStaticAssets`. Посмотрим, чем он отличается от стандартного `UseStaticFiles`, который применялся в раннем `AspNetCore`.

```C#
app.MapStaticAssets();
```

Прежде всего `UseStaticFiles` обслуживает статические файлы, но не обеспечивает тот
же уровень оптимизации, что и `MapStaticAssets`. `MapStaticAssets` оптимизирован для обслуживания ресурсов, о которых приложение знает во время выполнения. Но если приложение обслуживает ресурсы из других мест следует использовать `UseStaticFiles`.

Компонент `MapStaticAssets` предоставляет следующие преимущества: Сжатие во время сборки для всех ресурсов в приложении, включая `JavaScript` и `CSS`, но исключая ресурсы изображений и шрифтов, которые уже сжаты. Во время разработки применяется `Gzip`, 
а во время публикации в режиме релизовой сборки  приложения сжатие `Brotli`.

Для всех ресурсов во время сборки приложения применяется отпечаток `Fingerprinting` с помощью строки хэша `SHA-256` для содержимого каждого файла в кодировке `Base64`. Это предотвращает повторное использование старой версии файла, даже если старый файл кэширован. Статические ресурсы с отпечатком кэшируются с помощью `immutable`, что приводит к тому, что браузер никогда не запрашивает ресурс снова, пока он не изменится. Для браузеров, которые не поддерживают `immutable`, добавляется директива `max-age`.

Даже если ресурс не применяет отпечаток `Fingerprinting`, для каждого статического ресурса генерируется тег `ETag` на основе контента с использованием хэша `fingerprint` файла в качестве значения `ETag`. Это гарантирует отсутствие повторной загрузки.

Внутренне фреймворк сопоставляет физические ресурсы с их отпечатками `fingerprint`,
что позволяет приложению находить автоматически сгенерированные ресурсы, такие как `CSS` для определенных частей приложения. Кроме того, фреймворк может генерировать теги ссылок в элементе `<head>` страницы для предварительной загрузки ресурсов.

Во время тестирования разработки `VisualStudio` и использования `HotReload` информация о целостности удаляется из ресурсов, чтобы избежать проблем при изменении файла 
во время работы приложения, и статические ресурсы не кэшируются браузером.

При этом следует отметить, что `MapStaticAssets` не поддерживает ряд функций, 
которые поддерживаются `UseStaticFiles`, в частности: Обслуживание файлов с
диска или встроенных ресурсов или других расположений, Обслуживание файлов за пределами корневого каталога wwwroot, Установка заголовков ответа `HTTP`, Просмотр каталогов, Файлы по умолчанию, Обслуживание нескольких расположений файлов.

