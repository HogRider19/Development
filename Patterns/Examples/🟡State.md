**Состояние (State)** - шаблон проектирования, который позволяет объекту изменять свое поведение в зависимости от внутреннего состояния.

### Когда применяется State:

- Когда поведение объекта должно зависеть от его состояния и может изменяться динамически во время выполнения
- Когда в коде методов объекта используются многочисленные условные конструкции, выбор которых зависит от текущего состояния объекта

```c#
class Program
{
    static void Main()
    {
        Context context = new Context(new StateA());
        context.Request(); // Переход в состояние StateB
        context.Request();  // Переход в состояние StateA
    }
}

abstract class State
{
    public abstract void Handle(Context context);
}

class StateA : State
{
    public override void Handle(Context context)
    {
        context.State = new StateB();
    }
}

class StateB : State
{
    public override void Handle(Context context)
    { 
        context.State = new StateA();
    }
}
 
class Context
{
    public State State { get; set; }
    
    public Context(State state)
    {
        this.State = state;
    }
    
    public void Request()
    {
        this.State.Handle(this);
    }
}
```

[[🟡Behavioral]]