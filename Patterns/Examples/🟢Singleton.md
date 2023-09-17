**Одиночка (Singleton)** - порождающий паттерн, который гарантирует, что для определенного класса будет создан только один объект, а также предоставит к этому объекту точку доступа.

Когда надо использовать Singleton? Когда необходимо, чтобы для класса существовал только один экземпляр

Singleton позволяет создать объект только при его необходимости. Если объект не нужен, то он не будет создан. В этом отличие Singleton от глобальных переменных.

```c#
class Singleton
{
    private static Singleton _instance;
    private static object _lock = new Object();
 
    protected Singleton() {}
 
    public static Singleton Create()
    {
		lock (_lock)
		{
			if (_instance == null)
				_instance = new OS(name);
			return _instance;
		}
    }
}
```

[[🟡Generative]]