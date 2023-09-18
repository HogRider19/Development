**Интерпретатор (Interpreter)** - определяет представление грамматики для заданного языка и интерпретатор предложений этого языка. Как правило, данный шаблон проектирования применяется для часто повторяющихся операций.

```c#
class Client
{
    void Main()
    {
        Context context = new Context();
        var expression = new NonterminalExpression();
        expression.Interpret(context);
    }
}
 
class Context {}
 
abstract class AbstractExpression
{
    public abstract void Interpret(Context context);
}
 
class TerminalExpression : AbstractExpression
{
    public override void Interpret(Context context) {}
}
 
class NonterminalExpression : AbstractExpression
{
    AbstractExpression expression1;
    AbstractExpression expression2;
    
    public override void Interpret(Context context) {}
}
```

[[🟡Behavioral]]