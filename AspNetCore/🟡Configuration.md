Важную роль в приложении играет конфигурация, которая определяет базовые настройки приложения. Приложение может получать конфигурационные настройки из следующих источников: Аргументы командной строки, Переменные среды окружения, Объекты в памяти, Файлы `JSON`, Сервера `Azure`, или также из пользовательских источников.
Конфигурация приложения представляет объект интерфейса `IConfiguration`.

```c#
public interface IConfiguration
{
    string this[string key] { get; set; }
    IEnumerable<IConfigurationSection> GetChildren();
    IConfigurationSection GetSection(string key);
    IChangeToken GetReloadToken();
}
```

Данный интерфейс во-первых предоставляет индексатор `this[string key]`, который позволяет получить значение параметра конфигурации по указанному ключу. При этом и ключ, и значение представлены объектами типа `string`. Во-вторых, метод `GetChildren()` возвращает набор подсекций текущей секции конфигурации в виде коллекции. Кроме того, метод `GetReloadToken()` позволяет получить объект `IChangeToken`, предназначенный для отслеживания изменений конфигурации. Наконец, С помощью метода `GetSection(key)` можно получить конкретную секцию конфигурации, соответствующую заданному ключу. Также `GetConnectionString(name)` позволяет получить строку подключения по заданному имени, что особенно полезно при работе с базами данных и другими внешними ресурсами.

Также конфигурация может быть представлена интерфейсом `IConfigurationRoot`, который наследуется от IConfiguration и представляет агрегирующую точку получения конфигураций.

```C#
public interface IConfigurationRoot : IConfiguration
{
    IEnumerable<IConfigurationProvider> Providers { get; }
    void Reload();
}
```

В приложении настройки конфигурации хранятся в свойстве `Configuration` объекта `WebApplication`. Через это свойство мы можем установить или получить все настройки.
У приложение это свойство представлено `IConfiguration`, а у билдера `IConfigurationRoot`.

```c#
app.Configuration["name"] = "Tom";
app.Configuration["age"] = "100";
 
app.Run(async (context) =>
{
    var config = app.Services.GetRequiredService<IConfiguration>();
    await context.Response.WriteAsync($"{config["name"]} - {config["age"]}");
});
```

В примере выше настройки конфигурации устанавливались обычным словарем. Однако если настроек много или если они имеют сложную структуру, гораздо проще установить их одним скопом, особенно в случае, когда настройки хранятся в файле` json` или берутся из другого источника конфигурации. Для добавления источника конфигурации в приложении можно применять свойство `Configuration` билдера. Оно представляет `ConfigurationManager`,
наследника `IConfigurationRoot` для него определены методы добавления источников.

```c#
builder.Configuration.AddInMemoryCollection(
	new Dictionary<string, string>
{
    {"name", "Tom"},
    {"age", "37"}
});
 
var app = builder.Build();

app.Run(async (context) =>
{
    var config = app.Services.GetRequiredService<IConfiguration>();
    await context.Response.WriteAsync($"{config["name"]} - {config["age"]}");
});
```

По умолчанию `AspNetCore` сначала в качестве провайдера конфигурации устанавливается объект класса `ChainedConfigurationProvider`, который добавляет объект `IConfiguration` в качестве источника конфигурации и который фактически соединяет все далее применяемые провайдеры конфигурации в одну цепочку. Далее загружается конфиг из `appsettings.json`. Затем загружается конфигурация из файла `appsettings.[Environment].json`, где передает название окружения, например, `appsettings.Production.json` , благодаря чему мы можем задать конфигурацию для разных состояний проекта. Если проект запущен в окружении с именем "Development", загружается конфигурация `AppSecrets` для получения данных из операционной системы. Затем загружаются переменные и аргументы командной строки.

---

В `AspNetCore` параметры конфигурации можно загружать из различных источников c помощью провайдеров конфигурации: аргументы командной строки, переменных среды, переменные в памяти приложения, а также из различных файлов `JSON`, `XML` и `INI`. 

**==CommandLine==**. Aргументы командной строки автоматически добавляются в объект конфигурации. Обычно нет необходимости явно подключать их с помощью метода `AddCommandLine()`, так как они уже передаются через параметр `args` в статический 
метод создания билдера в формате словаря `WebApplication.CreateBuilder(args)`.
Aргументы можно задать в файле `launchSettings.json` в секции профиля запуска.

```c#
string[] commandLineArgs = { "name=Sam", "age=25" };
builder.Configuration.AddCommandLine(commandLineArgs);
```

```json
"HelloApp": {
	"commandLineArgs": "name=Bob age=37",
	"commandName": "Project",
	"launchBrowser": true,
},
```

**==EnvVars==**. Для загрузки переменных среды окружения в качестве параметров конфигурации применяется метод `AddEnvironmentVariables()`. Однако в реальности вряд ли его придется часто использовать, так как среда `AspNetCore` уже загружает переменные среды окружения.

```C#
builder.Configuration.AddEnvironmentVariables();
```

**==Memory==**. Провайдер `MemoryConfigurationProvider` позволяет использовать в качестве конфигурации коллекцию `IEnumerable<KeyValuePair<string, string>>`, которая хранит данные в виде пары ключ значение. Для добавления источника конфигурации применяется метод `AddInMemoryCollection()`, в который передаются конфигурационные настройки.

```c#
builder.Configuration.AddInMemoryCollection(new Dictionary<string, string>
{
    {"name", "Tom"},
    {"age", "37"}
});
```

**==Json==**. Как правило, для хранения конфигурации в приложении `AspNetCore` используются файлы `json`. Для работы с файлами `json` применяется `JsonConfigurationProvider`, а
для загрузки конфигурации из `json` применяется метод расширения `AddJsonFile()`.

По умолчанию в проекте уже есть файл конфигурации `appsettings.json`, а также `appsettings.Development.json`, которые загружаются по умолчанию в приложении
и которые мы можем использовать для хранения конфигурационных настроек.

Если подключается несколько провайдеров конфигурации с одинаковыми настройками, 
то параметры из источника, добавленного позже, будут переопределять ранние источники.

```c#
builder.Configuration.AddJsonFile("config.json");
```

---

Если конфигурационный файл содержит иерархию настроек, то получение дочерних значений происходит через обращение к элементу `IConfiguration` по ключу, где
полный ключ составлен из пути ключей до значения, разделенных двоеточием.

```c# 
app.Configuration["section1:section2:value"]
```

Но такой способ не всегда удобный, намного правильнее обращаться к секциям конфигурации через метод `GetSection(name)` интерфейса `IConfiguration`. 

Каждая отдельная секция представляет объект `IConfigurationSection`. Если секция содержит другие секции, то у нее также можем вызвать у ней метод `GetSection()`. 
Если же секция содержит только значение, то оно доступно через свойство `Value`.

Также есть интерфейс `IConfigurationSection` определяет  `GetConnectionString(name)` эквивалентный простому вызову метода `GetSection("ConnectionStrings")[name]`.

Этот метод никогда не возвращает `null`. Если соответствующий подраздел с указанным ключом не найден в конфигурации, возвращается пустой `IConfigurationSection` раздел.

```c#
app.Map("/", (IConfiguration appConfig) =>
{
    IConfigurationSection connStrings = appConfig.GetSection("ConnectionStrings");
    string defaultConnection1 = connStrings.GetSection("DefaultConnection").Value;
    string defaultConnection2 = appConfig["ConnectionStrings:DefaultConnection"];
    string defaultConnection3 = appConfig.GetConnectionString("DefaultConnection");
});
```

У интерфейса конфигурации и всех его секций есть метод `Get<T>()`, который позволяет создавать объекты с спроецированными на них данными конфигурации через рефлексию.

```json
{
  "SiteSettings": {
    "SiteTitle": "My Simple Site",
    "PageSize": 10
  }
}
```

```c#
public class SiteSettings
{
    public string SiteTitle { get; set; }
    public int PageSize { get; set; }
}
```

```c#
var site = builder.Configuration.GetSection("SiteSettings").Get<SiteSettings>();
```

---

Фреймворк `AspNetCore` позволяет проецировать конфигурационные настройки на классы.
Например, определим файл `person.json`, который будет хранить данные пользователя:

```json
{
  "name": "Tom",
  "age": "22"
}
```

```c#
public class Person
{
    public string Name { get; set; } = "";
    public int Age { get; set; } = 0;
}
```

```c#
var tom = new Person();
builder.Configuration.AddJsonFile("person.json");
app.Configuration.Bind(tom);
 
app.Run(async (context) => 
{
	await context.Response.WriteAsync($"{tom.Name} - {tom.Age}");
});
```

Для объекта `IConfiguration` определен метод `Bind()`, который в качестве параметра принимает объект, который надо связать с данными. Между конфигурацией в `json` и классом `Person` имеется соответствие по названию свойств, благодаря чему может осуществляться связка, регистр в данном случае роли не имеет никакого значения.
В качестве альтернативы методу `Bind` мы могли бы использовать метод `Get<T>()`.
В примерах выше выполнялась привязка корневого объекта конфигурации, 
однако также можно осуществлять привязку отдельных секций настроек.

---

Фреймворк реализует паттерн `Options`, который позволяет передавать конфигурацию 
не просто как набор настроек в виде пар ключей, а как объекты определенных классов.

Для применения этого паттерна в приложении у объекта `IServiceCollection`, который представляет коллекцию сервисов в приложения, определен его метод `Configure()`.

Этот метод реализован как метод расширения для типа `IServiceCollection`. Он типизируются типом, объект которого надо передавать через механизм внедрения зависимостей. И в качестве параметра принимает объект с типом `IConfigration`.

И также все версии метода принимают в качестве одного из параметров объект конфигурации, на основе которой будет создаваться объект с типом `TOptions`.

После чего через механизм внедрения зависимостей мы можем получить созданный объект через сервис с типом `IOptions<T>`, которые типизируется тем же типом, что и `Configure`.

```c#
builder.Services.Configure<Person>(builder.Configuration);
app.Map("/", (IOptions<Person> options) => options.Value);
```

Также метод `Configure` имеет ряд перегрузок, которые позволяют переопределять ранее установленные параметры. Одна из таких перегрузок принимает делегат, который на вход получает экземпляр типизированного объекта для изменения исходной конфигурации.

```c#
builder.Services.Configure<Person>(builder.Configuration);
builder.Services.Configure<Person>(opt => opt.Age = 22);
```

```json
{
  "age": "37",
  "name": "Tom",
  "languages": [
    "English",
    "German",
    "Spanish"
  ],
}
```

```c#
public class Person
{
    public string Name { get; set; } = "";
    public int Age { get; set; }
    public List<string> Languages { get; set; } = new();
}
```

```c#
builder.Configuration.AddJsonFile("person.json");
builder.Services.Configure<Person>(builder.Configuration);

var app = builder.Build();
 
app.Map("/", (IOptions<Person> options) =>
{
    Person person = options.Value;
});
```

---

Фреймворк `AspNetCore` по умолчанию предоставляет богатый функционал для работы с конфигурацией. Однако, в каких-то ситуациях этого может быть недостаточно. И в этом случае мы можем определить свои источники и провайдеры конфигурации данных.

Создание конфигурации вовлекает три компонента: `IConfigurationSource`, который определяет источник конфигурации, `ConfigurationProvider` представляющий сам провайдер и некий класс, который добавляет метод расширения к `IConfiguration`.

Допустим, мы хотим хранить конфигурацию в простом текстовом файле. И для
этого добавим новый класс, который назовем `TextConfigurationProvider`:

```c#
public class TextConfigurationProvider : ConfigurationProvider
{
    public string FilePath { get; set; }
    
    public TextConfigurationProvider(string path)
    {
        FilePath = path;
    }
    
    public override void Load()
    {
        var data = new Dictionary<string, string>();
        using (StreamReader textReader = new StreamReader(FilePath))
        {
            string? line;
            while ((line = textReader.ReadLine()) != null)
            {
                string key = line.Trim();
                string? value = textReader.ReadLine() ?? "";
                data.Add(key, value);
            }
        }
        
        Data = data;
    }
}
```

Класс `TextConfigurationProvider` будет представлять провайдер конфигурации 
и поэтому должен быть унаследован от базового класса `ConfigurationProvider`.

В этом классе с помощью `StreamReader` происходит считывание текстового файла и добавление данных в словарь `Data`. Для загрузки данных переопределяется `Load()`.

 Cвойство `Data` как раз и хранит все те конфигурационные настройки, которые потом используются в программе. Далее нам надо обращаться к этому провайдеру. Для этого определим класс источника конфигурации, который назовем `TextConfigurationSource`:

```c#
public class TextConfigurationSource : IConfigurationSource
{
    public string FilePath { get; }
    
    public TextConfigurationSource(string filename)
    {
        FilePath = filename;
    }
    
    public IConfigurationProvider Build(IConfigurationBuilder builder)
    {
        string filePath = builder.GetFileProvider()
	        .GetFileInfo(FilePath).PhysicalPath;
        return new TextConfigurationProvider(filePath);
    }
}
```

Источник конфигурации должен реализовать интерфейс `IConfigurationSource`,
и в частности, его метод `Build`. В этот метод передается строитель конфигурации.

В данном случае этот объект нам позволяет получить полный путь к текстовому файлу. Относительный путь передается в класс источника через конструктор и хранится в `FilePath`. После создания полного пути к файлу этот путь передается в`TextConfigurationProvider`.
Чтобы задействовать функционал источника создадим класс `TextConfigurationExtensions`:

```c#
public static class TextConfigurationExtensions
{
    public static IConfigurationBuilder AddTextFile(
        this IConfigurationBuilder builder, string path)
    {
        if (builder == null)
        {
            throw new ArgumentNullException(nameof(builder));
        }
        
        if (string.IsNullOrEmpty(path))
        {
            throw new ArgumentException("Путь к файлу не указан");
        }
 
        var source = new TextConfigurationSource(path);
        builder.Add(source);
        return builder;
    }
}
```

Этот класс определяет для объекта `IConfigurationBuilder` метод `AddTextFile()`, в котором создается источник конфигурации, который затем добавляется к строителю конфигурации.

```c#
var app = builder.Build();
builder.Configuration.AddTextFile("config.txt");
```

[[AspNetCore/🟡Base|🟡Base]]
