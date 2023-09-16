**Task RequestDelegate(HttpContext context, next)** - делегат middleware

## Inline Middleware:

---

**app.Run(RedDel)** - добавляет финальный middleware

**app.Use(RedDel)** - добавляет middleware с вызова следующего **await next.Invoke(context)** - передача контекста следующему middeware

**app.Map(path, builder=>)** - применяется для создания ветки конвейера, которая будет обрабатывать запрос по определенному пути

**app.UseWhen(context=>bool, buider=>)** - на основании некоторого условия позволяет создать ответвление конвейера при обработке запроса

**app.MapWhen(context=>bool, buider=>)** - аналогичен UseWhen, но в отличии от него не может передавать контекст в основную ветку

---

## Class Middleware:

---

**app.UseMiddleware()** - добавление middleware

### Класс middleware должен иметь:

1. Конструктор , который принимает RequestDelegate (next)
2. InvokeAsync, который принимает HttpContext