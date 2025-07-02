Важное место в приложении занимает аутентификация и авторизация. Аутентификация представляет процесс определения пользователя. Авторизация представляет процесс определения, имеет ли пользователь право доступа к некоторому ресурсу. Система
`AspNetCore` имеет поддержку аутентификации и авторизации пользователей.

![[Pasted image 20250702220444.png]]

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

Для создания объекта `Claim` определено множество конструкторов, но чаще
всего применяется следующая версия конструктора принимающая два параметра:

```c#
public Claim(string type, string value)
```

В качестве первого параметра в конструктор передается тип `Claim` это некоторая строка, которая, описывает назначение `Claim`. В качестве второго параметра передается значение.
В качестве типов можно использовать встроенные константы, к примеру `ClaimTypes.Name`, которая имеет значение `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` и которая обычно применяется для установки имени пользователя то, что потом мы сможем получить через свойство `HttpContext.User.Identity.Name`. Но при этом в качестве ключа типа мы можем использовать простые строки, но использование `ClaimTypes` позволяет получить внутреннюю поддержку записи свойств со стороны фреймворка `AspNetCore`.

```c#
var usernameClaim = new Claim(ClaimTypes.Name, "Tom");
```

Все объекты `Claim`, которые мы создали для описания пользователя, затем 
можно передать в виде коллекции в конструктор класса `ClaimsIdentity`:

```c#
var usernameClaim = new Claim(ClaimTypes.Name, "Tom");
var claimsIdentity = new ClaimsIdentity(
	new List<Claim> { usernameClaim }, "Cookies");
```

Объект `ClaimsIdentity` определяет ряд вспомогательных методов, которые
позволяют удобно манипулировать внутренней коллекцией с утверждениями.

```c#
// 1. Создаем ClaimsIdentity и добавляем claims
var identity = new ClaimsIdentity();

// Добавляем одиночный claim
identity.AddClaim(new Claim(ClaimTypes.Name, "Иван Иванов"));

// Добавляем несколько claims
identity.AddClaims(new List<Claim>
{
	new Claim(ClaimTypes.Email, "ivan@example.com"),
	new Claim(ClaimTypes.Role, "Admin"),
	new Claim(ClaimTypes.Role, "Developer"),
	new Claim("CustomClaim", "CustomValue"),
	new Claim(ClaimTypes.DateOfBirth, "1985-05-15", ClaimValueTypes.Date)
};);

// 2. Создаем ClaimsPrincipal
var principal = new ClaimsPrincipal(identity);

// 3. Работа с Claims через ClaimsPrincipal
Console.WriteLine("Все claims через Principal:");
foreach (var claim in principal.Claims)
	Console.WriteLine($"{claim.Type}: {claim.Value}");

// 4. Поиск claims
var roleClaims = principal.FindAll(ClaimTypes.Role);
foreach (var claim in roleClaims)
	Console.WriteLine(claim.Value);

var firstEmail = principal.FindFirst(ClaimTypes.Email);
Console.WriteLine($"\nПервый email: {firstEmail?.Value}");

// 5. Проверка наличия claim
bool hasAdminRole = principal.HasClaim(ClaimTypes.Role, "Admin");
Console.WriteLine($"\nЕсть ли роль Admin: {hasAdminRole}");

bool hasCustomClaim = principal.HasClaim(c => 
	c.Type == "CustomClaim" && c.Value == "CustomValue");

// 6. Проверка роли
bool isInAdminRole = principal.IsInRole("Admin");
Console.WriteLine($"Принадлежит ли к роли Admin: {isInAdminRole}");

// 7. Работа непосредственно с ClaimsIdentity
Console.WriteLine("\nРабота с ClaimsIdentity:");

// Поиск claims в identity
var dobClaim = identity.FindFirst(ClaimTypes.DateOfBirth);
Console.WriteLine($"Дата рождения: {dobClaim?.Value}");

// Проверка наличия claim
bool hasDobClaim = identity.HasClaim(c => c.Type == ClaimTypes.DateOfBirth);
Console.WriteLine($"Есть ли claim с датой рождения: {hasDobClaim}");

// Удаление claim
var claimToRemove = identity.FindFirst("CustomClaim");
if (claimToRemove != null)
{
	identity.RemoveClaim(claimToRemove);
	Console.WriteLine("CustomClaim был удален");
}

// Попытка удаления с TryRemoveClaim (в .NET Core 3.0+)
var emailClaim = new Claim(ClaimTypes.Email, "ivan@example.com");
bool removed = identity.TryRemoveClaim(emailClaim);
Console.WriteLine($"Email claim был удален: {removed}");

// Вывод оставшихся claims
Console.WriteLine("\nОставшиеся claims:");
foreach (var claim in identity.Claims)
	Console.WriteLine($"{claim.Type}: {claim.Value}");
```

Если мы динамически решим добавить новый `claim` или удалить существующий, то
после изменения `claim` необходимо заново пересоздавать объект `ClaimsPrincipal`
и перезаписывать аутентификационные куки или `jwt` токен, где эти данные храниться.

```c#
app.MapGet("/addage", async (HttpContext context) =>
{
    if(context.User.Identity is ClaimsIdentity claimsIdentity)
    {
        claimsIdentity.AddClaim(new Claim("age", "37"));
        var claimsPrincipal = new ClaimsPrincipal(claimsIdentity);
        await context.SignInAsync(claimsPrincipal);
    }
    return Results.Redirect("/");
});
```

---

Авторизация по ролям позволяет разграничить доступ к ресурсам приложения в зависимости от роли, к которой принадлежит пользователь данного приложения. 

Для указания роли здесь применяется тип `Claim` `ClaimsIdentity.DefaultRoleClaimType`, 
а в качестве значения для этого типа используется имя роли. По сути больше для установки роли в приложении для пользователя никаких других утверждений больше не нужно.

Чтобы на уровне отдельных ресурсов приложения разграничить доступ в зависимости
от роли, свойству `Roles` атрибута `Authorize` передается набор допустимых ролей:

```c#
app.Map("/admin", [Authorize(Roles = "admin")]() => "Admin Panel");

app.Map("/", [Authorize(Roles = "admin, user")](HttpContext context) =>
{
    var login = context.User.FindFirst(ClaimsIdentity.DefaultNameClaimType);
    var role = context.User.FindFirst(ClaimsIdentity.DefaultRoleClaimType);  
    return $"Name: {login?.Value}\nRole: {role?.Value}";
});
```

---

Атрибут `Authorize` легко позволяет разграничить доступ в зависимости от роли, однако для создания авторизации функциональности ролей бывает недостаточно. Например, что если мы хотим разграничить доступ на основе возраста пользователя или  других признаков.

Для этого применяется авторизация на основе `Claims`. Собственно авторизация на основе ролей фактически представляет частный случай авторизации на основе `Claims`, так как
роль это тот же объект `Claim`, имеющий тип `ClaimsIdentity.DefaultRoleClaimType`.

Для авторизации на основе `Claims` используются политики `Policy`. Политика представляет набор ограничений, которым должен соответствовать пользователь для доступа к ресурсу.

Все применяемые политики добавляются методом `builder.Services.AddAuthorization()`. Этот метод устанавливает политики с помощью объекта `AuthorizationOptions`. Например:

```c#
builder.Services.AddAuthorization(opts => {
     
    opts.AddPolicy("OnlyForMicrosoft", policy => {
        policy.RequireClaim("company", "Microsoft");
    });
});

app.Map("/", [Authorize(Policy = "OnlyForMicrosoft")](HttpContext context)
	=> return $"{context.User.FindFirst("company")}";);
```

В данном случае добавляется политика с именем `OnlyForMicrosoft`. И она требует обязательной установки для текущего пользователя объекта `Claim` с типом `Company`
и значением `Microsoft`. Если для пользователя не будет установлено подобного
объекта `Claim`, то такой пользователь не будет соответствовать этой политике.

```c#
// RequireRole(roles) - пользователь должен иметь одну из ролей.
// RequireUserName(name) - пользователь должен иметь логин name.
// RequireAuthenticatedUser() - пользователь должен быть аутентифицирован.
// RequireClaim(type) - для пользователя должен быть установлен claim с типом type.
// RequireClaim(type, values) - должен быть установлен с типом type и значением. 
// RequireAssertion(handler) - запрос должен соответствовать условию в делегате.
// AddRequirements(requirement) - позволяет добавить пользовательское ограничение.

services.AddAuthorization(options =>
{
	// 1. Настройка политики по умолчанию
	// Используется, когда [Authorize] применяется без параметров
	options.DefaultPolicy = new AuthorizationPolicyBuilder()
		.RequireAuthenticatedUser() // Требуется аутентификация
		.Build();
		
	// 2. Политика для администраторов
	options.AddPolicy("AdminOnly", policy => 
	{
		policy.RequireAuthenticatedUser()
		  .RequireRole("Administrator"); // Требуется роль Administrator
	});
	
	// 3. Политика для редакторов контента
	options.AddPolicy("ContentEditors", policy =>
	{
		policy.RequireAuthenticatedUser()
		  .RequireRole("Editor", "SeniorEditor") // Для Editor или SeniorEditor
		  .RequireClaim("Department", "Content"); // И claim Department=Content
	});
	
	// 4. Политика для доступа к финансовым данным
	options.AddPolicy("FinanceAccess", policy =>
	{
		policy.RequireAuthenticatedUser()
		  .RequireClaim("Level", "High", "VeryHigh") // Level=High или VeryHigh
		  .RequireClaim("AccessArea", "Finance"); // И AccessArea=Finance
	});
	
	// 5. Политика для конкретного пользователя (например, системного аккаунта)
	options.AddPolicy("SystemAccount", policy =>
	{
		policy.RequireUserName("system@example.com"); // Только для этого логина
	});
	
	// 6. Политика с кастомной логикой проверки
	options.AddPolicy("MinimumExperience", policy =>
	{
		policy.RequireAuthenticatedUser()
			  .RequireAssertion(context =>
			  {
				  // Проверяем, что у пользователя есть claim с опытом
				  var experienceClaim = context.User.FindFirst("ExperienceYears");
				  if (experienceClaim == null) return false;
				  
				  // Парсим значение и проверяем, что опыт >= 3 лет
				  if (int.TryParse(experienceClaim.Value, out int years))
				  {
					  return years >= 3;
				  }
				  return false;
			  });
	});
	
	
	// 7. Политика с кастомным требованием
	options.AddPolicy("HasSecurityTraining", policy =>
	{
		policy.RequireAuthenticatedUser()
			  .AddRequirements(new CompletedTrainingRequirement("Security"));
			   // Кастомное требование
	});
});

// Регистрация обработчика для кастомного требования
services.AddSingleton<IAuthorizationHandler, CompletedTrainingHandler>();
```

---
Одним из подходов к авторизации и аутентификации в `AspNetCore` представляет механизм аутентификации и авторизации с помощью `JWT` токенов. Токен `JWT` представляет собой веб стандарт, который определяет способ передачи зашифрованных данных в формате `JSON`.

Для использования `JWT` токенов в проекте `AspNetCore` необходимо добавить `Nuget` пакет `Microsoft.AspNetCore.Authentication.JwtBearer`. Это можно сделать с помощью `NetCLI`.

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
    var claims = new List<Claim> { new Claim(ClaimTypes.Name, username) };
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

Константа `ISSUER` представляет издателя токена. Здесь можно определить любое название.
Константа `AUDIENCE` представляет потребителя токена, опять же может быть любая строка, обычно это сайт, на котором применяется токен. Константа `KEY` хранит ключ, который будет применяться для создания токена. Стоит отметить, что для разных алгоритмов шифрования могут применяться ограничения на размер ключа. Так, в данном примере применяется алгоритм `SecurityAlgorithms.HmacSha256`, для которого необходим ключ `32` байта.

И метод `GetSymmetricSecurityKey()` возвращает ключ безопасности, который применяется для генерации токена. Для генерации токена нам необходим объект класса SecurityKey. В качестве такого здесь выступает объект производного класса SymmetricSecurityKey, в конструктор которого передается массив байт, созданный по секретному ключу.

Чтобы указать, что приложение для аутентификации будет использовать токена, в метод `AddAuthentication` передается значение `JwtBearerDefaults.AuthenticationScheme`.

---

Одним из распространённых подходов к аутентификации и авторизации в `AspNetCore` является использование механизма `Cookie`. При этом после успешной аутентификации пользователю выдается `Cookie` с токеном, который автоматически передается браузером при каждом запросе к серверу. `Cookie` можно настраивать: задавать срок жизни, политику безопасности, имя и другие параметры. Этот подход удобен для классических приложений, где сервер и клиент работают в рамках одного домена. Для использования аутентификации по `Cookie` необходимо установить соответствующую схему в методе `AddAuthentication` и настроить параметры аутентификации пользователей через специальный метод `AddCookie`. 

В случае аутентификации по `Cookie` данные пользователя не хранятся на сервере. 
Вместо этого в `Cookie` записывается зашифрованный и подписанный токен с данными пользователя. Этот токен автоматически передается клиентом при каждом запросе.

```c#
builder.Services.AddAuthentication(
	CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
{
	// Путь перенаправления для не аутентифицированного пользователя
	options.LoginPath = "/login"; 
	
	// Путь перенаправления для аутентифицированного пользователя,
	// который не имеет прав для доступа к ресурсу
	options.AccessDeniedPath = "/accessdenied"; 
	
	// Время жизни cookie (по умолчанию — до закрытия браузера)
	options.ExpireTimeSpan = TimeSpan.FromDays(30); 
	
	// Позволяет обновлять время жизни cookie при каждом запросе
	options.SlidingExpiration = true; 
	
	// Имя cookie (по умолчанию ".AspNetCore.Cookies")
	options.Cookie.Name = "auth_cookie"; 
	
	// Домен, для которого будет действовать cookie 
	// options.Cookie.Domain = "example.com"; 
	// Путь, для которого cookie будет действителен (по умолчанию "/")
	options.Cookie.Path = "/"; 
	
	// Флаг, указывающий, что cookie доступен только через HTTP (без JS)
	options.Cookie.HttpOnly = true; 
	
	// Политика безопасности cookie (SameSiteMode.Strict/Lax/None)
	options.Cookie.SameSite = SameSiteMode.Strict; 
	
	// Требует HTTPS для передачи cookie (в Production рекомендуется true)
	options.Cookie.SecurePolicy = CookieSecurePolicy.Always; 
	
	// Позволяет обрабатывать событие перенаправления на логин
	options.Events.OnRedirectToLogin = context =>
	{
		// Кастомная обработка (например, для API возвращать 401 вместо редиректа)
		return Task.CompletedTask;
	};
	
	// Позволяет обрабатывать событие отказа в доступе
	options.Events.OnRedirectToAccessDenied = context =>
	{
		// Кастомная обработка (например, для API возвращать 403 вместо редиректа)
		return Task.CompletedTask;
	};
});
```

```c#
app.UseAuthentication();
app.UseAuthorization();

app.MapPost("/login", async (HttpContext context, string username) =>
{
    var claims = new List<Claim> { new Claim(ClaimTypes.Name, username)};
    var claimsIdentity = new ClaimsIdentity(claims, 
	    CookieAuthenticationDefaults.AuthenticationScheme);
    var authProperties = new AuthenticationProperties
    {
        IsPersistent = true, // cookie сохраняется между сессиями
        ExpiresUtc = DateTimeOffset.UtcNow.AddDays(30)
    };

    await context.SignInAsync(
        CookieAuthenticationDefaults.AuthenticationScheme,
        new ClaimsPrincipal(claimsIdentity),
        authProperties);

    return Results.Ok(new { message = "Успешный вход" });
});
```

[[AspNetCore/🟡Base|🟡Base]]
