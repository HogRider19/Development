Все данные запроса передаются в middleware через объект `HttpContext`. Этот объект инкапсулирует информацию о запросе, позволяет управлять ответом и так далее. 

**Microsoft.AspNetCore.Http.HttpContext** - объект инкапсулирующий запрос и ответ клиенту для передачи его по цепочке конвейеров в `MiddleWare`.

Для создания компонентов middleware используется делегат `RequestDelegate`, который выполняет некоторое действие и принимает контекст запроса - объект `HttpContext`. Контекст запроса инкапсулирует в себе информацию о забросе в виде свойств:

**Connection** - представляет информацию о подключении данного запроса.
**Features** - получает коллекцию HTTP функциональности текущего запроса.
**Items** - получает или устанавливает коллекцию пар для хранения некоторых данных.
**Request** - возвращает объект HttpRequest, который хранит информацию о запросе.
**RequestServices** - объект IServiceProvider для работы с зарегистрированными сервисами.
**Response** - возвращает объект HttpResponse, который позволяет управлять ответом.
**TraceIdentifier** - представляет идентификатор запроса для логов трассировки.
**WebSockets** - возвращает объект для управления подключениями WebSocket.
**User** - представляет пользователя, ассоциированного с этим запросом.
**Session** - хранит данные сессии для текущего запроса.

Для получения информации о запросе и управления ответом, который будет отправлен клиенту, контекст запроса имеет два свойства `Request` и `Response`.

---

**==Response==**. Свойство объекта `HttpContext`, представляет объект `HttpResponse` и задает, что будет отравляться в виде ответа. Для этого `HttpResponse` определяет свойства:

**Body** - получает или устанавливает тело ответа в виде объекта Stream.
**BodyWriter** - возвращает объект типа PipeWriter для записи ответа.
**HasStarted** - возвращает true, если отправка ответа уже началась.

**ContentLength** - получает или устанавливает заголовок Content-Length.
**ContentType** - получает или устанавливает заголовок Content-Type.
**StatusCode** - возвращает или устанавливает статусный код ответа.
**Host** - получает или устанавливает заголовок Host.

**Headers** - возвращает заголовки ответа.
**Cookies** - возвращает куки, отправляемые в ответе.
**HttpContext** - возвращает контекст, связанный с данным ответом.

Для установки заголовков применяется свойство `Headers`, которое представляет тип `IHeaderDictionary`. Для большинства стандартных заголовков в этом интерфейсе определены одноименные свойства. Другие можно добавить через метод `Append()`.

Чтобы отправить ответ, мы можем использовать ряд методов класса `HttpResponse`:

**Redirect()** - выполняет переадресацию (временную или постоянную) на другой ресурс.
**WriteAsJson()** - отправляет ответ в виде объектов в формате JSON.
**WriteAsync()** - отправляет некоторое содержимое в виде текста. 
**SendFileAsync()** - отправляет файл.

Для отправки файлов применяется метод `SendFileAsync()`, который получает либо путь к файлу в виде строки, либо информацию о файле в виде объекта `IFileInfo`.

Таким способом можно отправлять файлы с кодом разметки, или с постоянным для выдачи контентом, но стоит отметить, что в ASP.NET Core уже имеется встроенный middleware, который позволяет упростить работу со статическими файлами.

---

**==Request==**. Свойство `Request` объекта `HttpContext` представляет объект `HttpRequest` и хранит информацию о запросе, отправленного клиентом, в виде следующих свойств:

**Body** - предоставляет тело запроса в виде объекта Stream.
**BodyReader** - возвращает объект типа PipeReader для чтения тела запроса.

**ContentLength** - получает или устанавливает заголовок Content-Length.
**ContentType** - получает или устанавливает заголовок Content-Type.
**Cookies** - возвращает коллекцию куки (объект Cookies).
**Headers** - возвращает заголовки запроса.
**Host** - получает или устанавливает заголовок Host.

**Form** - получает или устанавливает тело запроса в виде форм.
**HasFormContentType** - проверяет наличие заголовка Content-Type
**HttpContext** - возвращает связанный с данным запросом HttpContext

**IsHttps** - возвращает true, если применяется протокол https.
**Method** - получает или устанавливает метод HTTP.
**Path** - путь запроса в виде объекта RequestPath.
**PathBase** - получает или устанавливает базовый путь запроса.
**Protocol** - получает или устанавливает протокол, например, HTTP.
**Query** - возвращает коллекцию параметров из строки запроса.
**QueryString** - получает или устанавливает строку запроса.
**RouteValues** - получает данные маршрута для текущего запроса.
**Scheme** - получает или устанавливает схему запроса HTTP.

Нередко данные отправляются на сервер с помощью форм, обычно в запросе типа POST. Для получения таких данных в классе `HttpRequest` определено свойство `Form`.

Свойство `Request.Form` возвращает объект `IFormCollection`, где по ключу можно получить значение элемента. В качестве ключей выступает названия полей формы.

[[AspNetCore/🟡Base|🟡Base]]

