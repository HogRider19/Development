---

---
Основным элементом в архитектуре `AspNetCoreMvc` является контроллер. При получении запроса система маршрутизации выбирает для обработки запроса нужный контроллер и передает данные запроса. Контроллер обрабатывает его и посылает обратно результат.

При использовании контроллеров существуют некоторые условности. Обычно 
в проекте контроллеры помещаются в каталог `Controllers`. Однако это в принципе необязательно можно добавлять контроллеры в другие папки или даже в корень проекта.

В `AspNetCoreMvc` контроллер представляет обычный класс на языке `C#`, который обычно наследуется от абстрактного базового класса `Microsoft.AspNetCore.Mvc.Controller` и который, как и любой класс на языке `C#`, может иметь поля, свойства, методы. Согласно соглашениям об именовании названия контроллеров обычно оканчиваются на суффикс `Controller`, остальная же часть до этого суффикса считается именем контроллера, например, `HomeController`. Но в принципе эти условности также необязательны.

Но есть также и обязательные условности, которые предъявляются к контроллерам. В частности, класс контроллера должен удовлетворять как минимум одному из условий:

```c#
// 1) Класс контроллера имеет суффикс "Controller"
public class HomeController { }

// 2) Класс контроллера наследуется от класса, который имеет суффикс "Controller"
public class Home : Controller { }

// 3) К классу контроллера применяется атрибут [Controller]
[Controller]
public class Home { }
```

Ключевым элементом контроллера являются его действия. Действия контроллера представляют собой публичные методы, которые могут сопоставляться с запросами. Например, возьмем контроллер `HomeController` и определим в нем метод `Index`:

```c#
public class HomeController : Controller
{
	public string Index()
	{
		return "Hello METANIT.COM";
	}
}
```

Сопоставление запроса с контроллером и его действием происходит благодаря системе маршрутизации. И для настройки сопоставления используется метод `MapControllerRoute`":

```c#
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

В качестве шаблона маршрута применяется `{controller=Home}/{action=Index}/{id?}`, который представляет трехсегментный запрос. В нем первый сегмент представляет контроллер, второй сегмент метод контроллера, а третий необязательный параметр.
При этом если в запросе не указаны сегменты например, обращение идет к корню приложения, то тогда в качестве контроллера применяется `HomeController/Index`.

Собственно поэтому система может соотнести запрос типа `localhost:xxxx/Home/Index`
с контроллером и его действием. Чтобы обратиться контроллеру из веб-браузера, нам
надо в адресной строке набрать `адрес_сайта/Имя_контроллера/Действие_контроллера`. 

Однако не все методы контроллера являются действиями. Контроллер также может иметь непубличные методы  такие методы не рассматриваются системой как действия контроллера.

Возможно, сопоставление по умолчанию бывает не всегда удобно. Например, у нас есть класс в папке `Controllers`, но мы не хотим, чтобы он мог обрабатывать запрос. Чтобы указать, что этот класс не является контроллером, нам нужен атрибут `[NonController]`.

```c#
[NonController]
public class HomeController : Controller { }
```

Аналогично, если мы хотим, чтобы какой-либо публичный метод контроллера не рассматривался как действие, то мы можем использовать атрибут `[NonAction]`
Атрибут `[ActionName]` позволяет для метода задать другое имя действия:

```c#
[NonAction]
public string Hello() => "Hello ASP.NET";

[ActionName("Welcome")]
public string Hello() => "Hello ASP.NET";
```

Кроме того, методы могут обслуживать разные типы запросов. Для указания типа запроса `HTTP` нам надо применить к методу один из атрибутов: `[HttpGet]`, `[HttpPost]`, `[HttpPut]`, `[HttpPatch]`, `[HttpDelete]` и `[HttpHead]`. Если атрибут явным образом не указан, то метод может обрабатывать все перечисленные типы `Http` запросов: `GET`, `POST`, `PUT`, `DELETE`.

```c#
public class HomeController : Controller
{
	[HttpGet]
	public string Index() => "Hello METANIT.COM";
	
	[HttpPost]
	public string Hello() => "Hello ASP.NET";
}
```

---

При обращении к контроллеру среда `AspNetCore` создает для этого контроллера контекст, который содержит различные связанные с контроллером данные. Для получения контекста
в классе контроллера нам доступно свойство `ControllerContext`, которое представляет одноименный класс `ControllerContext`. Этот класс определяет ряд важный свойств:

```c#
public IActionResult Index()
{
	// Содержит информацию о контексте запроса
	var httpContext = ControllerContext.HttpContext;
	
	// Возвращает дескриптор действия - объект ActionDescriptor,
	// который описывает вызываемое действие контроллера
	var actionDescriptor = ControllerContext.ActionDescriptor;
	
	// Возвращает словарь ModelStateDictionary, который
	// используется для валидации данных, отправленных пользователем
	var modelState = ControllerContext.ModelState;
	
	// Возвращает данные маршрута
	var routeData = ControllerContext.RouteData;
	
	return View();
}

public abstract class ControllerBase  
{
	public HttpContext HttpContext => ControllerContext.HttpContext;
    public HttpRequest Request => HttpContext?.Request!;
    public HttpResponse Response => HttpContext?.Response!;
    public RouteData RouteData => ControllerContext.RouteData;
    public ModelStateDictionary ModelState => ControllerContext.ModelState;
}
```

Действие контроллера по факту представляет собой конечную точку. Из-за этого действие контроллера позволяет записывать результат с использованием подхода через `Responce`.

---

Вместе с запросом приложению могут приходить различные данные. И чтобы получить эти данные, мы можем использовать разные способы. К примеру через параметры строки `URL`.
Определение в методах контроллера параметров ничем не отличается от определения параметров в языке C#. Например, определим в контроллере следующий метод:

```c#
public class HomeController : Controller
{
	public string Index(string name) => $"Your name: {name}";
}
```

Применение параметров запроса в действия контроллера ничем не отличается
от применения параметров в обычных контрольных точках чистого `AspNetCore`.

Параметры представляют самый простой способ получения данных, но в действительности нам необязательно их использовать. В контроллере доступен объект `Request`, у которого можно получить как данные строки запроса через свойство `Request.Query`. Это свойство представляет объект `IQueryCollection`, где по ключу названию можно получить значение.

```c#
public string Index()
{
    string name = Request.Query["name"];
    string age = Request.Query["age"];
    return $"Name: {name}  Age: {age}";
}
```

---

Кроме `GET` запросов также применяются `POST` запросы. Как правило, такие запросы отправляются с помощью форм на странице. Но основные принципы будут теми же.

```c#
public class HomeController : Controller
{
	[HttpGet]
	public async Task Index()
	{
		string content = @"<form method='post'>
			<label>Name:</label><br />
			<input name='name' /><br />
			<label>Age:</label><br />
			<input type='number' name='age' /><br />
			<input type='submit' value='Send' />
		</form>";
		Response.ContentType = "text/html;charset=utf-8";
		await Response.WriteAsync(content);
	}
	
	[HttpPost]
	public string Index(string name, int age) => $"{name}: {age}";
}
```

Первый метод `Index` имеет атрибут `[HttpGet]`, поэтому данный метод будет обрабатывать только запросы `GET`. Для упрощения примера в ответ метод будет возвращать `html` код с формой ввода хотя естественно, для формы `html` можно было бы определить страницу.

Таким образом, при обращении к методу пользователь увидит в браузере форму ввода. 
При нажатии на кнопку `Send` введенные данные будут отправляться на сервер. Поскольку
у элемента `<form>` не задан атрибут `action`, который устанавливает адрес, то введенные данные отправляются на тот же адрес. Но поскольку у формы установлен `method='post'`, 
то данные будут отправлять в запросе типа `POST`. Обрабатывать сего будет второй метод.

Данные в параметрах запроса, которые принимаются через форму, поддерживают те же способы передачи, что и обычные параметры обработчика: списки, словари и так далее.
Для явного указания, что параметр в обработчике запроса должен быть получен
из параметров формы, можно использовать специальный атрибут `[FromForm]`.

```c#
[HttpPost]
public string Index([FromForm] string name)
{
	return name;
}
```

Для получения данных отправленных форм можно использовать свойство `Request.Form`. Это свойство представляет коллекцию `IFormsCollection`, где каждый элемент имеет ключ
и значение. В качестве ключа выступает название поля формы, а в качестве значенияlданные.

```c#
[HttpPost]
public string PersonData()
{
	string name = Request.Form["name"];
	string age = Request.Form["age"];
	return $"{name}: {age}";
}
```

---

При обращении к приложению, как правило, пользователь ожидает получить некоторый ответ, например, в виде веб страницы, которая наполнена данными. На стороне сервера метод контроллера, получая параметры и данные запроса, обрабатывает их и формирует ответ в виде результата действия. Результат действия это возвращаемый действием объект.

В качестве результата действия может возвращаться практически что угодно: `void`, `string`, какой-нибудь сложный объект. Но в большинстве случаев мы будем иметь дело с объектами типа `IActionResult`, которые непосредственно предназначены для генерации результата действия. Интерфейс `IActionResult` определяет в себе лишь один единственный метод:

```c#
public interface IActionResult
{
    Task ExecuteResultAsync(ActionContext context);
}
```

Метод `ExecuteResultAsync()` принимает контекст действия и генерирует результат.
Этот интерфейс затем реализуется абстрактным базовым классом `ActionResult`:

```c#
public abstract class ActionResult : IActionResult
{
    public virtual Task ExecuteResultAsync(ActionContext context)
    {
		ExecuteResult(context);
		return Task.FromResult(true);
    }
    
    public virtual void ExecuteResult(ActionContext context) {}
}
```

Класс `ActionResult` добавляет синхронный метод, который выполняется в асинхронном.
И если мы вдруг захотим создать свой класс результата действий, то как раз можем либо унаследовать его от `ActionResult`, либо реализовать сам интерфейс `IActionResult`.

Нужно понимать, что `ActionResult` не связан с `IResult`, то есть с механизмом `ResultsApi`, который применяется для генерации результатов в контрольных точках `MinimalApi`. Для использования стандартных результатов определены наследники класса `ActionResult`:

```c#
// ContentResult - возврат строки
return Content("Hello world", "text/plain");

// EmptyResult - пустой ответ (200 OK)
return new EmptyResult();

// NoContentResult - пустой ответ (204 No Content) 
return NoContent();

// FileResult варианты
return File(fileBytes, "image/png", "image.png"); // FileContentResult
return File("~/files/doc.pdf", "application/pdf"); // VirtualFileResult
return PhysicalFile("/path/to/file.txt", "text/plain"); // PhysicalFileResult
return File(stream, "application/octet-stream"); // FileStreamResult

// ObjectResult - возврат объекта (сериализуется в JSON)
return new ObjectResult(new { Name = "John", Age = 30 });

// Статусные коды
return StatusCode(403); // StatusCodeResult
return Unauthorized(); // 401
return NotFound(); // 404
return BadRequest(); // 400
return Ok(); // 200

// С данными
return Ok(new { Id = 1 }); // OkObjectResult
return NotFound(new { Error = "Not found" }); // NotFoundObjectResult
return BadRequest(new { Error = "Invalid" }); // BadRequestObjectResult

// Результаты создания
return Created("/api/items/1", new { Id = 1 });
return CreatedAtAction("GetItem", new { id = 1 }, new { Id = 1 });
return CreatedAtRoute("itemRoute", new { id = 1 }, new { Id = 1 });

// Принятые запросы (202)
return Accepted(); // AcceptedResult
return Accepted("/api/status/123"); // AcceptedResult с location
return AcceptedAtAction("Status", new { id = 123 }); // AcceptedAtActionResult

// JSON
return Json(new { Value = 42 }); // JsonResult

// Перенаправления
return Redirect("/new-url"); // RedirectResult (302)
return RedirectPermanent("/permanent-url"); // 301
return LocalRedirect("/local-path"); // LocalRedirectResult
return RedirectToAction("Index"); // RedirectToActionResult
return RedirectToRoute("default", new { page = 1 }); // RedirectToRouteResult

// Представления
return View(); // ViewResult
return PartialView(); // PartialViewResult
return ViewComponent("MyComponent"); // ViewComponentResult

// Аутентификация
return Challenge(); // ChallengeResult
```

---

В контроллерах также работает механизм внедрения зависимостей, зависимости можно передавать через конструктор, через сервис локатор `HttpContext.RequestServices` или внедрять напрямую в методы, используя при этом специальный атрибут `FromService`.

```c#
public string Index([FromServices] ITimeService timeService)
{
    return timeService.Time;
}
```

---

Как правило, для создания контроллера достаточно унаследовать свой класс от базового класса `Controller`. Однако если нам необходимо, чтобы наши контроллеры реализовали некоторую общую логику, мы можем определить свой базовый класс контроллера и уже
от него наследовать остальные контроллеры. Либо мы также можем переопределить некоторые методы базового `Controller`, если реализации нас не устраивают.

```c#
// Метод вызывается перед выполнением действия контроллера
// Можно использовать для:
// - Проверки входных параметров
// - Логирования начала выполнения
// - Установки context.Result для отмены действия
[NonAction]
public virtual void OnActionExecuting(ActionExecutingContext context) { }

// Метод вызывается после выполнения действия контроллера
// Можно использовать для:
// - Обработки результатов
// - Логирования завершения
// - Модификации возвращаемых данных
[NonAction]
public virtual void OnActionExecuted(ActionExecutedContext context) { }

// Асинхронный обработчик, объединяющий логику до и после выполнения действия
// Работает по принципу:
// 1. Сначала выполняет OnActionExecuting
// 2. Если не было прерывания, вызывает следующее звено цепочки через next()
// 3. После выполнения вызывает OnActionExecuted с результатом
// Параметры:
// - context: контекст выполнения с информацией о запросе
// - next: делегат для вызова следующего обработчика или самого действия
[NonAction]
public virtual async Task OnActionExecutionAsync(
	ActionExecutingContext context,
	ActionExecutionDelegate next)
{
	if (context == null) throw new ArgumentNullException(nameof(context));
	if (next == null) throw new ArgumentNullException(nameof(next));
	
	OnActionExecuting(context);
	if (context.Result == null)
	{
	    OnActionExecuted(await next());
	}
}
```