**Наблюдатель (Observer)** - представляет поведенческий шаблон проектирования, который использует отношение "один ко многим". В этом отношении есть один наблюдаемый объект и множество наблюдателей. И при изменении наблюдаемого объекта автоматически происходит оповещение всех наблюдателей.

Данный паттерн еще называют Publisher-Subscriber (издатель-подписчик), поскольку отношения издателя и подписчиков характеризуют действие данного паттерна: подписчики подписываются email-рассылку определенного сайта. Сайт-издатель с помощью email-рассылки уведомляет всех подписчиков о изменениях. А подписчики получают изменения и производят определенные действия: могут зайти на сайт, могут проигнорировать уведомления и т.д.

Инфраструктура для реализации данного паттерна уже реализована в dotNet. И представлена интерфейсами  IObserver\<T> и IObservable\<T>. Также вместо паттерна Observer можно воспользоваться стандартным механизмом событий. 

### Когда использовать Observer:

- Когда система состоит из множества классов, объекты которых должны находиться в согласованных состояниях
- Когда общая схема взаимодействия объектов предполагает две стороны: одна рассылает сообщения и является главным, другая получает сообщения и реагирует на них. Отделение логики обеих сторон позволяет их рассматривать независимо и использовать отдельно друга от друга.
- Когда существует один объект, рассылающий сообщения, и множество подписчиков, которые получают сообщения. При этом точное число подписчиков заранее неизвестно и процессе работы программы может изменяться.

```c#
interface IObservable
{
    void AddObserver(IObserver o);
    void RemoveObserver(IObserver o);
    void NotifyObservers();
}

class ConcreteObservable : IObservable
{
    private List<IObserver> observers;
    
    public ConcreteObservable()
    {
        observers = new List<IObserver>();
    }
    
    public void AddObserver(IObserver o)
    {
        observers.Add(o);
    }
 
    public void RemoveObserver(IObserver o)
    {
        observers.Remove(o);
    }
 
    public void NotifyObservers()
    {
        foreach (IObserver observer in observers)
            observer.Update();
    }
}
 
interface IObserver
{
    void Update();
}

class ConcreteObserver :IObserver
{
    public void Update() {}
}
```

[[🟡Behavioral]]