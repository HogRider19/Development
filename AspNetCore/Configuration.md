В приложении настройки конфигурации хранятся в свойстве Configuration объекта WebApplication. Соответственно через это свойство мы можем установить или получить настройки конфигурации

**IConfiguration** - интерфейс определяющий конфигурацию приложения **IConfigurationRoot** - интерфейс для агрегатора конфигураций **ConfigurationSection** - интерфейс секции конфигурации

## Интерфейс IConfiguration:

1. **GetSection(string key)** - возвращает секцию конфигурации, которая соответствует ключу key
2. **GetChildren()** - возвращает набор подсекций текущей секции конфигурации в виде объекта IEnumerable
3. **GetConnectionString(name)** - получение значения строки подключения name из cсекции ConnectionStrings
4. **GetReloadToken()** - возвращает объект IChangeToken, который применяется для отслеживания изменения конфигурации

---

**JsonConfigurationProvider** - провайдер JSON конфигурации **XmlConfigurationProvide**r - провайдер XML конфигурации **IniConfigurationProvider** - провайдер ini конфигурации

**Config.AddJsonFile()** - метод расширения для загрузки json **Config.AddXmlFile()** - метод расширения для загрузки xml **Config.AddIniFile()** - метод расширения для загрузки ini **Config.AddInMemoryCollection()** - конфигурация из словаря