По умолчанию веб-браузеры в целях безопасности ограничивают `ajax` запросы между различными доменами. Однако нередко возникает ситуация, когда необходимо выполнять запросы из приложения с одного адреса к приложению, которое размещено по другому адресу. Для этого нам надо использовать технику `CORS` `Cross Origin Resource Sharing`.

Для подключения сервисов `CORS` вызывается метод `builder.Services.AddCors()`.
Чтобы задействовать `CORS` для обработки запроса вызывается метод `app.UseCors()`.
Для конфигурации параметров `CORS` этот метод использует делегат, в который передается объект `CorsPolicyBuilder`. И с помощью этого объекта можно выполнить настройку `CORS`.

```c#
var builder = WebApplication.CreateBuilder();
builder.Services.AddCors();
var app = builder.Build();
 
// настраиваем CORS
app.UseCors(builder => builder.AllowAnyOrigin());
app.Map("/", async context => await context.Response.WriteAsync("Hello!"));
app.Run();
```

```c#
services.AddCors(options =>
{
	options.AddPolicy("MyCorsPolicy", builder =>
	{
		builder
			// Разрешаем запросы с определённых источников
			.WithOrigins("https://example.com", "https://another-example.com")
			// Разрешаем только определённые методы
			.WithMethods("GET", "POST", "PUT")
			// Разрешаем только определённые заголовки
			.WithHeaders("Content-Type", "Authorization")
			// Разрешаем отправку куки и других учётных данных
			.AllowCredentials()
			// Указываем заголовки, которые клиент сможет прочитать в ответе
			.WithExposedHeaders("X-Custom-Header", "X-Another-Header");
	});

	// Альтернативная политика для полного доступа
	options.AddPolicy("AllowAllPolicy", builder =>
	{
		builder
			.AllowAnyOrigin()
			.AllowAnyMethod()
			.AllowAnyHeader();
		// AllowAnyOrigin и AllowCredentials не могут использоваться вместе!
	});
```