**Команда (Command)** - позволяет инкапсулировать запрос на выполнение определенного действия в виде отдельного объекта. Этот объект запроса на действие и называется командой. При этом объекты, инициирующие запросы на выполнение действия, отделяются от объектов, которые выполняют это действие.

Команды могут использовать параметры, которые передают ассоциированную с командой информацию. Кроме того, команды могут ставиться в очередь и также могут быть отменены.

### Когда использовать Command:

- Когда надо передавать в качестве параметров определенные действия, вызываемые в ответ на другие действия. То есть когда необходимы функции обратного действия в ответ на определенные действия.
- Когда необходимо обеспечить выполнение очереди запросов, а также их отмену.
- Когда надо поддерживать логгирование изменений в результате запросов. Использование логов может помочь восстановить состояние системы - для этого необходимо будет использовать последовательность запротоколированных команд.

---

Таким образом, инициатор, отправляющий запрос, ничего не знает о получателе, который и будет выполнять команду. Кроме того, если нам потребуется применить какие-то новые команды, мы можем просто унаследовать классы от абстрактного класса Command и реализовать его методы Execute и Undo.

В программах на C# команды находят довольно широкое применение. Так, в технологии WPF и других технологиях, которые используют XAML и подход MVVM, на командах во многом базируется взаимодействие с пользователем. В некоторых архитектурах, например, в архитектуре CQRS, команды являются одним из ключевых компонентов.

```c# 
abstract class Command
{
    public abstract void Execute();
    public abstract void Undo();
}

class ConcreteCommand : Command
{
    Receiver receiver;
    
    public ConcreteCommand(Receiver r)
    {
        receiver = r;
    }
    
    public override void Execute()
    {
        receiver.Operation();
    }
 
    public override void Undo() {}
}
 
class Receiver
{
    public void Operation() {}
}

class Invoker
{
    Command command;
    
    public void SetCommand(Command c)
    {
        command = c;
    }
    
    public void Run()
    {
        command.Execute();
    }
    
    public void Cancel()
    {
        command.Undo();
    }
}

class Client
{  
    void Main()
    {
        Invoker invoker = new Invoker();
        Receiver receiver = new Receiver();
        ConcreteCommand command = new ConcreteCommand(receiver);
        invoker.SetCommand(command);
        invoker.Run();
    }
}
```

[[🟡Behavioral]]