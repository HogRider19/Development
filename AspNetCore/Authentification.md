Важное место в приложении занимает аутентификация и авторизация. Аутентификация представляет процесс определения пользователя. Авторизация представляет процесс определения, имеет ли пользователь право доступа к некоторому ресурсу. Система
`AspNetCore` имеет поддержку аутентификации и авторизации пользователей.

---

Для выполнения аутентификации в конвейере обработки запроса отвечает специальный компонент `middleware`  `AuthenticationMiddleware`. Для встраивания этого `middleware` в конвейер применяется специальный метод расширения системы `UseAuthentication()`.

Следует отметить, что метод `UseAuthentication()` должен встраиваться в конвейер до любых компонентов `middleware`, которые используют аутентификацию пользователей.
Для выполнения аутентификации этот компонент использует сервисы аутентификации, в частности `IAuthenticationService`, которые регистрируются через `AddAuthentication()`.

```c#
var builder = WebApplication.CreateBuilder();
builder.Services.AddAuthentication("Bearer") 
    .AddJwtBearer();
 
var app = builder.Build();
app.UseAuthentication();
```

В качестве параметра вторая версия метода `AddAuthentication()` принимает схему аутентификации в виде строки. Третья версия метода `AddAuthentication` принимает
делегат, который устанавливает опции аутентификации объект `AuthenticationOptions`.

Какую бы мы версию метода не использовали, для аутентификации необходима установить схему аутентификации. Часто используемые представлены в виде строк `Cookies` и `Bearer`.
Одна хранит идентификатор в куках, а вторая записывает его в `jwt` токен пользователя.    

Схема аутентификации позволяет выбирать определенный обработчик аутентификации. Обработчик аутентификации собственно и выполняет непосредственную аутентификацию пользователей на основе данных запросов и исходя из выбранной схемы аутентификации.

Например, для аутентификации с помощью куки передается схема `Cookies`.
Соответственно для аутентификации пользователя будет выбираться встроенный обработчик аутентификации `Authentication.Cookies.CookieAuthenticationHandler`, который на основе полученных в запросе `cookie` выполняет аутентификацию.

А если используется схема `Bearer`, то это значит, что будет использоваться `jwt` токен, а
в качестве обработчика применяться класс `Authentication.JwtBearer.JwtBearerHandler`. Стоит отметить, что для аутентификации с помощью `jwt` токенов необходимо добавить в проект через `Nuget` сторонний пакет `Microsoft.AspNetCore.Authentication.JwtBearer`.

При чем в `AspNetCore` мы не ограничены встроенными схемами аутентификации и 
можем создавать свои собственные и под них своих обработчиков аутентификации.
Кроме применения схемы аутентификации необходимо подключить аутентификацию определенного типа. Для этого можно использовать следующие методы расширения:

```c#
builder.Services.AddAuthentication("Bearer").AddJwtBearer();
builder.Services.AddAuthentication("Cookies").AddCookie();
```

Оба метода реализованы как методы расширения для `AuthenticationBuilder`, 
который возвращается методом добавления сервисов `AddAuthentication()`.

---

Авторизация представляет процесс определения прав пользователя в системе,
к каким ресурсам приложения он имеет право доступа и при каких условиях.

Хотя авторизация представляет отдельный независимый процесс, тем не менее 
для нее также необходимо, чтобы приложение также применяло аутентификацию.

Для подключения авторизации необходимо встроить компонент `AuthorizationMiddleware`. Для этого применяется встроенный метод расширения приложения `UseAuthorization()`.
Также нужно зарегистрировать сервисы авторизации с помощью `AddAuthorization()`.

Вторая версия метода принимает делегат, который с помощью специального
объекта `AuthorizationOptions` позволяет сконфигурировать авторизацию. 

Ключевым элементом механизма авторизации в `AspNetCore` является атрибут `AuthorizeAttribute` , который позволяет ограничить доступ к ресурсам.

```c#
var builder = WebApplication.CreateBuilder();
 
builder.Services.AddAuthentication("Bearer").AddJwtBearer();    
builder.Services.AddAuthorization();           
 
var app = builder.Build();
 
app.UseAuthentication();
app.UseAuthorization();
 
app.Map("/hello", [Authorize]() => "Hello World!");
app.Map("/", () => "Home Page");
 
app.Run();
```

---
Одной из задач аутентификации в приложении `AspNetCore` является установка пользователя, который представлен в виде свойства `User` класса `HttpContext`.

```c#
public abstract System.Security.Claims.ClaimsPrincipal User { get; set; }
```

Данное свойство предоставляет класс `ClaimsPrincipal`. Непосредственно данные, которые идентифицируют пользователя хранятся в свойстве `Identity` класса `ClaimPrincipal`.

```c#
public virtual System.Security.Principal.IIdentity? Identity { get; }
```

Это свойство представляет основную идентичность. Но так как с  пользователем может
быть связан набор идентичностей, то также в классе определено свойство `Identities`.

```c#
public virtual IEnumerable<ClaimsIdentity> Identities { get; }
```

Свойство `Identity` представляет интерфейс `IIdentity`, и, как правило, в качестве такой реализации применяется класс `ClaimsIdentity`. Объект `IIdentity`, в свою очередь, предоставляет информацию о текущем пользователе через следующие свойства:

```c#
public interface IIdentity
{
	// Возвращает имя пользователя. Обычно в качестве подобного имени
	// используется логин, по которому пользователь входит в приложение
	string? Name { get; }
	
	// Тип аутентификации в строковом виде
	string? AuthenticationType { get; }
	
	// Возвращает `true`, если пользователь аутентифицирован
	bool IsAuthenticated { get; }
}
```

Для создания объекта `ClaimsIdentity` можно применять ряд конструкторов, но, для того, чтобы пользователь был аутентифицирован, необходимо, как минимум, предоставить тип аутентификации, которая передается через конструктор. Тип аутентификации представляет произвольную строку, которая описывает некоторым образом способ аутентификации.

```c#
var identity = new ClaimsIdentity("Cookies");
var principal = new ClaimsPrincipal(identity);

await context.SignInAsync(claimsPrincipal);
await context.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
```

Аутентификацию пользователя можно произвести с помощью метода `context.SignInAsync`.
А разаутентифицировать пользователя можно через `context.SignOutAsync` у контекста.

```c#
builder.Services.AddAuthentication(
	CookieAuthenticationDefaults.AuthenticationScheme).AddCookie();
 
var app = builder.Build();
 
app.UseAuthentication();
 
app.MapGet("/login", async (HttpContext context) =>
{
    var claimsIdentity = new ClaimsIdentity("Undefined");
    var claimsPrincipal = new ClaimsPrincipal(claimsIdentity);
    
    // установка аутентификационных куки
    await context.SignInAsync(claimsPrincipal);
    return Results.Redirect("/");
});
 
app.MapGet("/logout", async (HttpContext context) =>
{
    await context.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
    return "Данные удалены";
});

app.Map("/", (HttpContext context) =>
{
    var user = context.User.Identity;
    if (user is not null && user.IsAuthenticated)
    {
        return $"Пользователь аутентифицирован";
    }
    else
    {
        return "Пользователь НЕ аутентифицирован";
    }
});
```

Текущего пользователя мы можем получить через свойство `context.User`. Фактически это тот самый объект `ClaimsPrincipal`, который мы записываем через `context.SignInAsync()`. И когда приходит запрос, инфраструктура `AspNetCore` дешифрует и десериализует данные запроса и создает по ним объект `ClaimsPrincipal`, который хранится в `context.User`.

Если используется аутентификация на основе `Cookies`, то данные о пользователе будут извлекаться из аутентификационных кук. Если применяются `jwt` токены, то данные берутся из полученного токена. Причем даже если аутентификационных куки или токена в запросе нет, то объект `ClaimsPrincipal` все равно будет создаваться инфраструктурой `AspNetCore`.

Поскольку объект `HttpContext` доступен через механизм внедрения зависимостей в любой точке приложения, то мы можем через этот объект получить пользователя, как в примере выше. Однако, если нам нужно только свойство `User`, а не весь объект `HttpContext`, то мы можем также через механизм внедрения зависимостей получить сервис `ClaimsPrincipal`.

```c#
app.Map("/", (ClaimsPrincipal claimsPrincipal) =>
{
    var user = claimsPrincipal.Identity;
    if (user is not null && user.IsAuthenticated) 
        return "Пользователь аутентифицирован";
    else return "Пользователь не аутентифицирован";
});
```

---

Ранее был рассмотрен объект `context.User`, который представляет класс `ClaimsPrincipal` и который хранит данные пользователя в свойстве `Identity` в виде `ClaimsIdentity`. Но что представляют сами данные пользователя? Для хранения данных определены объекты `Claim`.

Объекты `Claim` представляют некоторую информацию о пользователе, которую мы можем использовать для авторизации в приложении. У пользователя может быть определенный возраст, город, страна проживания, любимая музыкальная группа и прочие признаки. И все эти признаки могут представлять отдельные объекты `Claim`. И в зависимости от значения этих `Claim` мы можем предоставлять пользователю доступ к тому или иному ресурсу. Таким образом, `Claims` представляют более общий механизм авторизации нежели стандартные логины или роли, которые привязаны лишь к одному определенному признаку пользователя.

Каждый объект `Claim` представляет класс `Claim` из пространства имен `System.Security.Claims`, который определяет следующие свойства:

```C#
public class Claim  
{
	// Название системы или службы, которая выдала данный claim 
	public string Issuer { get; }
	
	// Исходный издатель claim, если он был перенаправлен или преобразован 
	// другими системами (например, при федеративной аутентификации)
	public string OriginalIssuer { get; }
	
	// Объект ClaimsIdentity, которому принадлежит данный claim
	// содержит информацию о пользователе и его идентичности
	public ClaimsIdentity? Subject { get; }
	
	// Тип данного claim, например, "name", "role", "email"
	// определяет, что означает значение claim
	public string Type { get; }
	
	// Основное значение claim, например, имя
	// пользователя, роль или адрес электронной почты
	public string Value { get; }
	
	// Тип данных значения, например, "http://www.w3.org/2001/XMLSchema#string" 
	// Оно помогает получателю claim понять, как интерпретировать его значение.
	public string ValueType { get; }
}
```


















---
Одним из подходов к авторизации и аутентификации в `AspNetCore` представляет механизм аутентификации и авторизации с помощью `JWT` токенов. Токен `JWT` представляет собой веб стандарт, который определяет способ передачи зашифрованных данных в формате `JSON`.

```c#
builder.Services.AddAuthorization();
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
.AddJwtBearer(options =>
{
	options.TokenValidationParameters = new TokenValidationParameters
	{
		// указывает, будет ли валидироваться издатель при валидации токена
		ValidateIssuer = true,
		// строка, представляющая издателя
		ValidIssuer = AuthOptions.ISSUER,
		// будет ли валидироваться потребитель токена
		ValidateAudience = true,
		// установка потребителя токена
		ValidAudience = AuthOptions.AUDIENCE,
		// будет ли валидироваться время существования
		ValidateLifetime = true,
		// установка ключа безопасности
		IssuerSigningKey = AuthOptions.GetSymmetricSecurityKey(),
		// валидация ключа безопасности
		ValidateIssuerSigningKey = true,
	 };
});
```

```c#
app.UseAuthentication();
app.UseAuthorization();
 
app.Map("/login/{username}", (string username) => 
{
    var claims = new List<Claim> {new Claim(ClaimTypes.Name, username) };
    var jwt = new JwtSecurityToken(
            issuer: AuthOptions.ISSUER,
            audience: AuthOptions.AUDIENCE,
            claims: claims,
            expires: DateTime.UtcNow.Add(TimeSpan.FromMinutes(2)),
            signingCredentials: 
	            new SigningCredentials(
		            AuthOptions.GetSymmetricSecurityKey(), 
		            SecurityAlgorithms.HmacSha256));
            
    return new JwtSecurityTokenHandler().WriteToken(jwt);
});
 
app.Map("/data", [Authorize] () => new { message= "Hello World!" });
```

```c#
public class AuthOptions
{
    public const string ISSUER = "MyAuthServer"; // издатель токена
    public const string AUDIENCE = "MyAuthClient"; // потребитель токена
    const string KEY = "mysupersecret_secretsecretsecretkey!123";   // ключ
    public static SymmetricSecurityKey GetSymmetricSecurityKey() => 
        new SymmetricSecurityKey(Encoding.UTF8.GetBytes(KEY));
}
```


































---
---
---

Важное место в приложении занимает аутентификация и авторизация. Аутентификация представляет процесс определения пользователя, а авторизация процесс определения, имеет ли пользователь право доступа к некоторому ресурсу. ASP.NET Core имеет встроенную поддержку как аутентификации так и авторизации.

---

Для выполнения аутентификации в конвейере обработки запроса определяется специальный компонент middleware `AuthenticationMiddleware`. Для встраивания этого компонента в конвейер применяется метод расширения `UseAuthentication()`.

Для выполнения аутентификации этот компонент использует сервисы аутентификации, в частности, сервис `IAuthenticationService`, которые регистрирует `AddAuthentication()`

В качестве параметра вторая версия метода `AddAuthentication()` принимает схему аутентификации в виде строки. Третья версия метода `AddAuthentication` принимает делегат, который устанавливает опции аутентификации - объект `AuthenticationOptions`.

Какую бы мы версию метода не использовали, для аутентификации необходима установить схему аутентификации. Две наиболее расcпространенные схемы аутентификации:` Cookies` на основе куков и `Bearer` на основе jwt токена.

Схема аутентификации позволяет выбирать определенный обработчик аутентификации. Обработчик аутентификации собственно и выполняет аутентификацию пользователей на основе данных запроса.

Например, для аутентификации с помощью куки будет выбираться встроенный обработчик `Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationHandler`, а для jwt токенов `Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler`.

При чем в ASP.NET Core мы не ограничены встроенными схемами аутентификации и можем создавать собственные схемы и под них своих обработчиков аутентификации.

Кроме применения схемы аутентификации необходимо подключить и сами сервисы аутентификации определенного типа. Они реализованы как методы расширения для типа `AuthenticationBuilder`, который возвращается методом `AddAuthentication()`:

**AddCookie()** - подключает и конфигурирует аутентификацию с помощью куки.
**AddJwtBearer()** - подключает и конфигурирует аутентификацию с помощью jwt-токенов (для этого метода необходим Nuget-пакет Microsoft.AspNetCore.Authentication.JwtBearer)

```c#
var builder = WebApplication.CreateBuilder();
builder.Services.AddAuthentication("Bearer").AddJwtBearer();

var app = builder.Build();
app.UseAuthentication();
```

---

Авторизация представляет процесс определения прав пользователя в системе, к каким ресурсам приложения он имеет право доступа и при каких условиях.

Хотя авторизация представляет отдельный независимый процесс, тем не менее для нее также необходимо, чтобы приложение также применяло аутентификацию.

Для подключения авторизации необходимо встроить `AuthorizationMiddleware`. Для этого применяется встроенный метод расширения UseAuthorization().

Кроме того, для применения авторизации необходимо зарегистрировать сервисы авторизации с помощью метода `AddAuthorization()`. Вторая версия метода принимает делегат, который с помощью `AuthorizationOptions` сконфигурирует авторизацию.

Ключевым элементом механизма авторизации является атрибут `AuthorizeAttribute`, который позволяет ограничить доступ к ресурсам приложения.

```c#
using Microsoft.AspNetCore.Authorization;
 
var builder = WebApplication.CreateBuilder();
 
builder.Services.AddAuthentication("Bearer") .AddJwtBearer(); 
builder.Services.AddAuthorization(); 
 
var app = builder.Build();
 
app.UseAuthentication(); 
app.UseAuthorization();  
 
app.Map("/hello", [Authorize]() => "Hello World!");
app.Map("/", () => "Home Page");
 
app.Run();
```

---

Одной из задач аутентификации в приложении является установка пользователя, который представлен в приложении свойством `User` класса `HttpContext`.
Данное свойство представляет из себя объект класса `ClaimsPrincipal`.

Непосредственно данные, которые идентифицируют пользователя хранятся в свойстве `Identity` класса `ClaimPrincipal`. Это свойство представляет основную идентичность текущего пользователя. Но поскольку с одним пользователем может быть связан набор идентичностей, то также в классе определено свойство `Identities`.

**HttpContext.User** - получение объекта ClaimsPrincipal текущего пользователя.
**ClaimsPrincipal.Identity** - получение объекта ClaimsIdentity текущего пользователя.

Свойство `Identity` представляет интерфейс `IIdentity`, и, как правило, в качестве такой реализации применяется класс `ClaimsIdentity`. Объект `IIdentity`, в свою очередь, предоставляет информацию о текущем пользователе через следующие свойства:

**AuthenticationType** - тип аутентификации в строковом виде.
**IsAuthenticated** - возвращает `true`, если пользователь аутентифицирован.
**Name** - возвращает имя пользователя, как правило логин пользователя.

Для создания объекта `ClaimsIdentity` можно применять ряд конструкторов, но, для того, чтобы пользователь был аутентифицирован, необходимо, как минимум, предоставить тип аутентификации, которая передается через конструктор. Тип аутентификации представляет строку, которая описывает способ аутентификации.

Для установки идентичности пользователя объект `ClaimsIdentity` можно передать в `ClaimsPrincipal` либо через конструктор, либо через метод `AddIdetity()`. После чего,  `ClaimsPrincipal` можно зарегистрировать через `context.SignInAsync(claimsPrincipal)`.

Для хранения различных данных пользователя во фреймворке определяются объекты `Claim`. Объекты `Claim` представляют некоторую информацию о пользователе, в свою очередь которую мы можем использовать для авторизации в приложении.

Все признаки могут представлять отдельные объекты `claim`. И в зависимости от их значений мы можем предоставлять пользователю доступ к тому или иному ресурсу. Таким образом, `claims` представляют более общий механизм авторизации нежели стандартные логины или роли, которые привязаны лишь к одному признаку.

Каждый объект `claim` представляет класс `Claim` из пространства имен `System.Security.Claims`, который определяет следующие свойства:

**Issuer** - издатель или название системы, которая выдала данный claim.
**Subject** - возвращает информацию о пользователе в виде объекта ClaimsIdentity.
**Type** - возвращает тип объекта claim.
**Value** - возвращает значение объекта claim.

Для создания объекта `Claim` применяются различные перегрузки его конструктора. Основной из которых принимает тип и значение устанавливаемого `Claim`.

После чего все объекты `Claim`, которые описывают пользователя, затем можно передать в виде коллекции в конструктор `ClaimsIdentity`.

![[Pasted image 20231231145729.png]]

Для работы с объектами `claim`, которые предоставляются всеми `ClaimsIdentity` пользователя, в экземпляре `ClaimsPrincipal` определены специальные методы:

**Claims** - свойство, которое возвращает набор всех объектов claim.
**FindAll(type/predicate)** - возвращает все объекты claim по условию или типу.
**FindFirst(type/predicate)** - возвращает первый объект claim по условию или типу.
**HasClaim(predicate)** - возвращает true, если пользователь имеет claim по условию.
**IsInRole(name)** - возвращает значение true, если пользователь принадлежит роли.

Более тонкое управление утверждениями осуществляется с помощью обращение к определенным методам и свойствам конкретного `ClaimsIdentity` пользователя:

**Claims** - свойство, которое возвращает набор всех объектов claim.
**FindAll(type/predicate)** - возвращает все объекты claim по условию или типу.
**FindFirst(type/predicate)** - возвращает первый объект claim по условию или типу.
**HasClaim(predicate)** - возвращает true, если пользователь имеет claim по условию.

**AddClaim(claim)** - добавляет для пользователя объект claim.
**AddClaims(claims)** - добавляет набор объектов claim.
**RemoveClaim(claim)** - удаляет объект claim у пользователя.
**TryRemoveClaim(claim)** - true если удалось удалить объект claim.

Если мы динамически решим добавить новый `claim` или удалить существующий, то после изменения `claim` необходимо заново пересоздавать объект `ClaimsPrincipal` и перезаписывать аутентификационные куки или jwt-токен, где эти данные хранятся.

```c#
app.MapGet("/login", async (HttpContext context) =>
{
    var claimsIdentity = new ClaimsIdentity("Undefined");
    var claimsPrincipal = new ClaimsPrincipal(claimsIdentity);
    
    await context.SignInAsync(claimsPrincipal);
    return Results.Redirect("/");
});
 
app.MapGet("/logout", async (HttpContext context) =>
{
    await context.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
    return "Данные удалены";
});

app.Map("/", (HttpContext context) =>
{
    var user = context.User.Identity;
    if (user is not null && user.IsAuthenticated)
    {
        return $"Тип аутентификации: {user.AuthenticationType}";
    }
    else
    {
        return "Пользователь НЕ аутентифицирован";
    }
});
```

---

Авторизация по ролям позволяет разграничить доступ к ресурсам приложения в зависимости от того к какой роли принадлежит данный пользователь.

Для осуществления механизма авторизации по ролям достаточно установить название роли в утверждение с типом `ClaimTypes.Role` или `ClaimsIdentity.DefaultRoleClaimType`.

После чего механизмом ролей пользователя можно убудет управлять через параметры атрибута `Authorize` при объявлении конечной точки маршрута в приложении. 

Атрибут `Authorize` легко позволяет разграничить доступ в зависимости от роли, однако для создания авторизации функциональности ролей бывает недостаточно. Например, что если мы хотим разграничить доступ на основе возраста пользователя.

Для этого применяется авторизация на основе `claims`. Собственно авторизация на основе ролей фактически представляет частный случай авторизации на основе `claims`.

Для авторизации на основе `claims` используются политики (`policy`). Политика представляет набор утверждений, которым должен соответствовать пользователь.

Все применяемые политики добавляются в приложение с помощью метода `builder.Services.AddAuthorization()`. Этот метод устанавливает политики:

```c#
builder.Services.AddAuthorization(opts => 
{
	opts.AddPolicy("OnlyForMicrosoft", policy => 
	{
		policy.RequireClaim("company", "Microsoft");
	});
});
```

Ключевым методом здесь является `AddPolicy()`. Первый параметр метода представляет название политики, а второй - делегат, который с помощью методов  объекта `AuthorizationPolicyBuilder` позволяет создать политику по условиям:

**RequireRole(roles)** - пользователь должен иметь одну из ролей.
**RequireUserName(name)** - пользователь должен иметь логин name.
**RequireAuthenticatedUser()** - пользователь должен быть аутентифицирован.
**RequireClaim(type)** - для пользователя должен быть установлен claim с типом type.
**RequireClaim(type, values)** -  должен быть установлен claim с типом type и значением. 
**RequireAssertion(handler)** - запрос должен соответствовать условию в делегате handler.
**AddRequirements(requirement)** - позволяет добавить пользовательское ограничение.

Хотя встроенный функционал по созданию политик авторизации покрывает множество случаев для их определения, но он имеет ограниченные возможности.

Класс ограничения должен реализовать интерфейс `IAuthorizationRequirement`. Сам класс ограничения только устанавливает некоторые лимиты, больше он ничего не делает. Чтобы его использовать, нам над добавить специальный класс обработчик.

Класс обработчика должен наследоваться от класса `AuthorizationHandler<TReq>`. Вся обработка производится в методе `HandleRequirementAsync()`. Этот метод вызывается системой авторизации при доступе к ресурсу, к которому применяется ограничение.

В качестве параметров метод `HandleRequirementAsync()` получает объект применяемого ограничения и контекст авторизации `AuthorizationHandlerContext`, который содержит информацию о запросе. В частности, через свойство `User` он возвращает объект `ClaimPrincipal`, представляющий текущего пользователя.

Методы класса `AuthorizationHandlerContext` позволяют управлять авторизацией. Так, метод `Succeed(requirement)` вызывается, если запрос соответствует ограничению requirement. И наоборот, метод `Fail()`, если запрос не соответствует ограничению.

**AuthorizationHandlerContext.Succeed()** - приводит к подтверждению ограничения.
**AuthorizationHandlerContext.Fail()** - приводит к отклонению ограничения.

[[AspNetCore/🟡Base|🟡Base]]
