Система определяет набор инструментов для упрощения отправки ответа `ResultsAPI`. Он реализуется посредством статических методов статического класса `Results`.

Этот механизм в большей степени предназначено для использования в конечных точках, но так же может применяться для отправки ответа и в компонентах middleware.

Все классы результатов реализуют один интерфейс `IResult`, который определяет один метод `Task ExecuteAsync(context)`. Его используют для создании своих `Results`.

Встроенные типы `ResultAPI` покрывают различные ситуации. Однако встроенных типов может оказаться недостаточно. Тогда мы можем определить свой тип `IResult`.

После чего создается метод расширения для интерфейса `IResultExtensions`, который принимает определенные параметры и возвращает созданный класс `IResult`. Данный метод будет доступен через свойство `Extentions` статического класса `Results`.

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
    {
	    return new HtmlResult(htmlCode);
    };
}

app.Map("/", () => Results.Extensions.Html("<h1>Hello</h1>"));
```

---

**Ok():** отправляет ответ со статусным кодом 200.
**BadRequest():** отправляет клиенту статусный код 400.
**NotFound():** отправляет ответ со статусным кодом 404.
**Created():** отправляет ответ со статусным кодом 201.
**NoContent():** отправляет статусный код 204.
**Conflict():** отправляет ответ со статусным кодом 409.
**Accepted():** отправляет клиенту статусный код 202.
**UnprocessableEntity():** отправляет ответ со статусным кодом 422.
**StatusCode():** отправляет определенный статусный код.

**Redirect():** перенаправляет на определенный адрес.
**LocalRedirect():** перенаправляет на один из локальных адресов.

**Problem():** применяет формат ProblemDetails для отправки ответа.
**Content():** отправляет в ответе некоторую строку.
**File():** отправляет в ответ файл.
**Stream():** пишет в ответ поток данных. аналогичен методу File.
**Bytes():** отправляет в ответ клиенту массив байтов.
**Json():** отправляет ответ в формате JSON.

**Challenge():** отправляет 401 или 403 (IAuthenticationService).
**Unauthorized():** отправляет статус код 401 (IAuthenticationService).
**Forbid():** отправляет в ответ клиенту 403 (IAuthenticationService).
**SignIn():** выполняет SignInAsync() (IAuthenticationService).
**SignOut():** выполняет SignOutAsync( (IAuthenticationService).

[[AspNetCore/Base|Base]]
