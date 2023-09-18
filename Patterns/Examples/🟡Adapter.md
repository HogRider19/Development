**Адаптер (Adapter)** - предназначен для преобразования интерфейса одного класса в интерфейс другого. Благодаря реализации данного паттерна мы можем использовать вместе классы с несовместимыми интерфейсами.

### Когда надо использовать Adapter:

- Когда необходимо использовать имеющийся класс, но его интерфейс не соответствует потребностям
- Когда надо использовать уже существующий класс совместно с другими классами, интерфейсы которых не совместимы

```c#
class Client
{
    public void Request(Target target)
    {
        target.Request();
    }
}

// класс, к которому надо адаптировать другой класс   
class Target
{
    public virtual void Request() {}
}
  
// Адаптер
class Adapter : Target
{
    private Adaptee adaptee = new Adaptee();
    
    public override void Request()
    {
        adaptee.SpecificRequest();
    }
}
  
// Адаптируемый класс
class Adaptee
{
    public void SpecificRequest() {}
}
```

[[🟡Structural]]