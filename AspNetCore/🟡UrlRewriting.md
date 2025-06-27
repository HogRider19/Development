Функциональность `UrlRewriting` позволяет контролировать доступ к определенным адресам в приложении. В частности, мы можем выполнить переопределение адресов, которые используются для доступа к ресурсам приложения. Например, `UrlRewriting` позволяет решить такие стандартные проблемы, как перенаправление с домена с `www`
на домен без `www` и наоборот или перенаправление с протокола `http` на `https`.

Сам `UrlRewriting` реализуется до того, как запрос попадет в систему маршрутизации,
и начнется его непосредственное выполнение в конвейере `MVC`. Запрошенный адрес изначально может отсутствовать в приложении, однако система может его изменить.

Для подключения `UrlRewriting` используется метод расширения `UseRewriter`, который принимает объект `RewriteOptions`, задающий все правила переопределения адресов:

```c#
using Microsoft.AspNetCore.Rewrite;
var builder = WebApplication.CreateBuilder();
var app = builder.Build();

app.UseRewriter(new RewriteOptions());
app.MapGet("/home", async context 
	=> await context.Response.WriteAsync("Hello World!"));
 
app.Run();
```

```c#
var options = new RewriteOptions();
    
// 1. AddRedirect - переадресация с кодом 301
options.AddRedirect("old-page", "new-page", (int)HttpStatusCode.MovedPermanently);

// 2. AddRewrite - подмена URL без перенаправления
options.AddRewrite(@"^rule/(.*)", "rewritten/$1", skipRemainingRules: true);

// 3. AddRedirectToWww - перенаправление на www поддомен (временное, 302)
options.AddRedirectToWww();

// 4. AddRedirectToWwwPermanent - перенаправление на www поддомен (постоянное, 301)
options.AddRedirectToWwwPermanent();

// 5. AddRedirectToNonWww - временное перенаправление с www на основной домен
options.AddRedirectToNonWww();

// 6. AddRedirectToNonWwwPermanent - перенаправление с www на основной домен
options.AddRedirectToNonWwwPermanent();

// 7. AddRedirectToHttps - перенаправление на HTTPS (временное, 302)
options.AddRedirectToHttps();

// 8. AddRedirectToHttpsPermanent - перенаправление на HTTPS (постоянное, 301)
options.AddRedirectToHttpsPermanent();

// 9. AddIISUrlRewrite - загрузка правил из IIS rewrite.xml
using (StreamReader iisUrlRewriteStreamReader = File.OpenText("IISUrlRewrite.xml"))
{
	options.AddIISUrlRewrite(iisUrlRewriteStreamReader);
}

// 10. AddApacheModRewrite - загрузка правил из .htaccess Apache
using (StreamReader apacheModRewriteStreamReader = File.OpenText(".htaccess"))
{
	options.AddApacheModRewrite(apacheModRewriteStreamReader);
}

app.UseRewriter(options);
```

[[AspNetCore/🟡Base|🟡Base]]
