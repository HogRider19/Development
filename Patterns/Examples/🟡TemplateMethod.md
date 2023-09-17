**Шаблонный метод (Template Method)** - определяет общий алгоритм поведения подклассов, позволяя им переопределить отдельные шаги этого алгоритма без изменения его структуры.

### Когда использовать шаблонный метод?

- Когда планируется, что в будущем подклассы должны будут переопределять различные этапы алгоритма без изменения его структуры
- Когда в классах, реализующим схожий алгоритм, происходит дублирование кода. Вынесение общего кода в шаблонный метод уменьшит его дублирование в подклассах.

```c#
abstract class AbstractClass
{
    public void TemplateMethod()
    {
        Operation1();
        Operation2();
    }
    
    public abstract void Operation1();
    public abstract void Operation2();
}
 
class ConcreteClass : AbstractClass
{
    public override void Operation1() {}
 
    public override void Operation2() {}
}
```
[[🟡Behavioral]]