**Фасад (Facade)** - представляет шаблон проектирования, который позволяет скрыть сложность системы с помощью предоставления упрощенного интерфейса для взаимодействия с ней.

### Когда использовать Facade:

- Когда имеется сложная система, и необходимо упростить с ней работу. Фасад позволит определить одну точку взаимодействия между клиентом и системой.
- Когда надо уменьшить количество зависимостей между клиентом и сложной системой. Фасадные объекты позволяют отделить, изолировать компоненты системы от клиента и развивать и работать с ними независимо.
- Когда нужно определить подсистемы компонентов в сложной системе. Создание фасадов для компонентов каждой отдельной подсистемы позволит упростить взаимодействие между ними и повысить их независимость друг от друга.

```c#
class SubsystemA
{
    public void A1() {}
}

class SubsystemB
{
    public void B1() {}
}

class SubsystemC
{
    public void C1() {}
}

public class Facade
{
    SubsystemA subsystemA;
    SubsystemB subsystemB;
    SubsystemC subsystemC;
 
    public Facade(SubsystemA sa, SubsystemB sb, SubsystemC sc)
    {
        subsystemA = sa;
        subsystemB = sb;
        subsystemC = sc;
    }
    
    public void Operation1()
    {
        subsystemA.A1();
        subsystemB.B1();
        subsystemC.C1();
    }
    
    public void Operation2()
    {
        subsystemB.B1();
        subsystemC.C1();
    }
}
 
class Client
{
    public void Main()
    {
        Facade facade = new Facade(
		    new SubsystemA(),
	        new SubsystemB(),
	        new SubsystemC()
        );
        
        facade.Operation1();
        facade.Operation2();
    }
}
```

[[🟡Structural]]