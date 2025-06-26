Система определяет набор инструментов упрощения отправки ответа `ResultsAPI`. 
Он реализуется посредством статических методов статического класса `Results`.

По средствам этого класса можно возвращать ответы пользователю из конечных
точек или даже из компонентов `middleware`. Все результаты реализуют `IResult`.

```c#
public interface IResult
{
	Task ExecuteAsync(HttpContext httpContext);
}
```

Хотя мы можем отправлять некоторый ответ напрямую из конечных точек, не используя методы `Results`, тем не менее методы `ResultsAPI` могут быть полезны, когда необходимо установить какие-то дополнительные параметры ответа: кодировку, форматирование `json`. В конечных точках можно использовать для отправки статусных кодов, файлов, редиректов.

```c#
app.Run(async context =>
{
    await Results.Text("Hello ASP.NET Core").ExecuteAsync(context);
});
```

Встроенные типы `ResultAPI` покрывают различные ситуации. Однако встроенных
типов может оказаться недостаточно. Тогда мы можем определить свой тип `IResult`.

После чего создается метод расширения для интерфейса `IResultExtensions`, который принимает определенные параметры и возвращает созданный класс `IResult`. Данный метод будет доступен через свойство `Extentions` для статического класса `Results`.

```c#
class HtmlResult: IResult
{
    string htmlCode = "";
    public HtmlResult(string htmlCode) => this.htmlCode = htmlCode;
 
    public async Task ExecuteAsync(HttpContext context)
    {
        context.Response.ContentType = "text/html; charset=utf-8";
        await context.Response.WriteAsync(htmlCode);
    }
}
 
static class ResultsHtmlExtension
{
    public static IResult Html(this IResultExtensions ext, string htmlCode)
	     => new HtmlResult(htmlCode);
}
```

---

```c#
// Возвращает успешный ответ с кодом `200 OK` и опциональным телом.
return Results.Ok(new { message = "Все прошло успешно!" });
```

```C#
// Возвращает ответ с кодом `400 Bad Request` и опциональным телом.
return Results.BadRequest("Неверные данные запроса.");
```

```C#
// Возвращает ответ `404 Not Found`, если ресурс не найден.
return Results.NotFound("Ресурс не найден.");
```

```C#
// Возвращает `201 Created` и заголовок `Location` с указанием URL нового ресурса.
var newItem = new { id = 123, name = "Item" };
return Results.Created($"/items/{newItem.id}", newItem);
```

```C#
// Возвращает ответ `204 No Content`, часто используется после успешного удаления.
return Results.NoContent()
```

```C#
// Возвращает `409 Conflict`, когда возникает конфликт с состоянием ресурса.
return Results.Conflict("Ресурс уже существует.");
```

```C#
// Возвращает `202 Accepted` — запрос принят, но не завершен.
return Results.Accepted("/status/123", new { status = "processing" });
```

```C#
// Возвращает `422`, когда сервер не может обработать содержимое запроса.
return Results.UnprocessableEntity("Ошибка валидации.");
```

```C#
// Позволяет вручную указать HTTP-статус код.
return Results.StatusCode(418);
```

```C#
// Перенаправляет клиента по абсолютному или относительному URL (302 по умолчанию).
return Results.Redirect("https://example.com");
```

```C#
// Безопасное перенаправление внутри приложения (только на локальные URL).
return Results.LocalRedirect("/new-location");
```

```C#
// Возвращает подробную ошибку в формате `ProblemDetails` (RFC 7807).
return Results.Problem(detail: "Ошибка", title: "Ошибка", statusCode: 500);
```

```C#
// Возвращает строку как `text/plain`, `text/html`, `application/json` и т.п.
return Results.Content("<h1>Hello</h1>", "text/html");
```

```C#
// Возвращает файл из файловой системы.
return Results.File("files/report.pdf", "application/pdf", "report.pdf");
```

```C#
// Отправляет поток (Stream) клиенту. Аналог `File()`, но для потоков.
var stream = new MemoryStream(Encoding.UTF8.GetBytes("Streamed content"));
return Results.Stream(stream, "text/plain");
```

```C#
// Возвращает массив байтов.
var data = Encoding.UTF8.GetBytes("Binary content");
return Results.Bytes(data, "application/octet-stream", "file.bin");
```

```C#
// Возвращает JSON-объект.
return Results.Json(new { message = "Hello JSON" });
```

```C#
// Отправляет ответ `401 Unauthorized`, запуская механизм аутентификации.
return Results.Challenge();
```

```C#
// Возвращает `401 Unauthorized`, без вызова middleware аутентификации.
return Results.Unauthorized();
```

```C#
// Возвращает `403 Forbidden`, указывая на отсуцтвие прав.
return Results.Forbid();
```

```C#
// Выполняет вход пользователя с помощью `SignInAsync`.
var claims = new List<Claim> { new Claim(ClaimTypes.Name, "user") };
var identity = new ClaimsIdentity(claims, "cookie");
var principal = new ClaimsPrincipal(identity);
await context.SignInAsync("cookie", principal);
return Results.Ok("Signed in");
```

```C#
// Выполняет выход пользователя с помощью `SignOutAsync`. 
await context.SignOutAsync("cookie");
return Results.Ok("Signed out");
```

[[AspNetCore/🟡Base|🟡Base]]
