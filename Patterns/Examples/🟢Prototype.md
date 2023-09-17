**Прототип (Prototype)** - позволяет создавать объекты на основе уже ранее созданных объектов-прототипов. То есть по сути данный паттерн предлагает технику клонирования объектов.

### Когда использовать Прототип?

- Когда конкретный тип создаваемого объекта должен определяться динамически
- Когда нежелательно создание отдельной иерархии классов фабрик для создания объектов-продуктов из параллельной иерархии классов (как это делается, например, при использовании паттерна Абстрактная фабрика)
- Когда клонирование объекта является более предпочтительным вариантом нежели его создание и инициализация с помощью конструктора. Особенно когда известно, что объект может принимать небольшое ограниченное число возможных состояний.


```c#
class Client
{
    public void Operation()
    {
        Prototype prototype = new ConcretePrototype1(1);
        Prototype clone = prototype.Clone();
        prototype = new ConcretePrototype2(2);
        clone = prototype.Clone();
    }
}
 
abstract class Prototype
{
    public int Id { get; private set; }
    
    public Prototype(int id)
    {
        this.Id = id;
    }
    
    public abstract Prototype Clone();
}
 
class ConcretePrototype1 : Prototype
{
    public ConcretePrototype1(int id) : base(id) { }
    
    public override Prototype Clone()
    {
        return new ConcretePrototype1(Id);
    }
}
 
class ConcretePrototype2 : Prototype
{
    public ConcretePrototype2(int id) : base(id) { }
    
    public override Prototype Clone()
    {
        return new ConcretePrototype2(Id);
    }
}
```
[[🟡Generative]]