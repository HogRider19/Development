**Стратегия (Strategy)** - представляет шаблон проектирования, который определяет набор алгоритмов, инкапсулирует каждый из них и обеспечивает их взаимозаменяемость. В зависимости от ситуации мы можем легко заменить один используемый алгоритм другим. При этом замена алгоритма происходит независимо от объекта, который использует данный алгоритм.

### Когда использовать Strategy:

- Когда есть несколько родственных классов, которые отличаются поведением. Можно задать один основной класс, а разные варианты поведения вынести в отдельные классы и при необходимости их применять
- Когда необходимо обеспечить выбор из нескольких вариантов алгоритмов, которые можно легко менять в зависимости от условий
- Когда необходимо менять поведение объектов на стадии выполнения программы
- Когда класс, применяющий определенную функциональность, ничего не должен знать о ее реализации

```c#
public interface IStrategy
{
    void Algorithm();
}
 
public class ConcreteStrategy1 : IStrategy
{
    public void Algorithm() {}
}
 
public class ConcreteStrategy2 : IStrategy
{
    public void Algorithm() {}
}
 
public class Context
{
    public IStrategy ContextStrategy { get; set; }
 
    public Context(IStrategy _strategy)
    {
        ContextStrategy = _strategy;
    }
 
    public void ExecuteAlgorithm()
    {
        ContextStrategy.Algorithm();
    }
}
```

[[🟡Behavioral]]