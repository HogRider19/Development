**Пул потоков** - это коллекция потоков, которые управляются системой CLR, чтобы оптимизировать выполнение многопоточных задач. Это уменьшает расходы на создание и уничтожение потоков, особенно при большом количестве коротких задач, поскольку потоки в пуле могут быть повторно использованы после завершения задачи.

При наличии большого количества методов, будет создан еще один фоновый поток. Когда поток отработает, он не уничтожается, а возвращается в пул поток для ожидания следующей задачи.

**Асинхронность** - это концепция выполнения программ, которая позволяет компьютеру одновременно выполнять несколько задач, не ожидая завершения каждой отдельной задачи. Это достигается путем выполнения задач в различных потоках или процессах. Также может быть реализована на возврате управления.

Асинхронность позволяет вынести отдельные задачи из основного потока в специальные асинхронные методы и при этом более экономно использовать потоки. 

Асинхронные методы выполняются в отдельных потоках. При выполнении продолжительной блокирующей операции поток асинхронного метода возвратится в пул потоков и будет использоваться для других задач.

Когда блокирующая операция завершается, для асинхронного метода опять выделяется поток из пула потоков, и асинхронный метод продолжает свою работу.

При работе с асинхронностью мы оперируем задачами. Это совсем не то же самое, что поток. Одна задача может выполняться многими потоками, а один поток может выполнять много задач.

---

Асинхронный метод преобразовывается компилятором в метод заглушку, в которой происходит инициализация сгенерированного класса - машины состояний. Далее машина состояний запускается, а из метода возвращается объект Тask.  
  
Особый интерес представляет метод _MoveNext_ машины состояний. Данный метод выполняет то, что было до преобразования в асинхронном методе. Он разбивает код на части между каждым вызовом await. 

Каждая часть выполняется при определенном состоянии машины. Сам метод MoveNext присоединяется к объекту ожидания в качестве продолжения. Сохранение состояния гарантирует выполнение части, которая логически следовала за ожиданием.

Асинхронная операция может пойти и по синхронному пути выполнения. Главное условие для того, чтобы текущий асинхронный метод выполнялся синхронно, то есть не меняя поток - это завершенность асинхронной операции до вызова await.

---

Для асинхронного выполнения метода, данный метод должен содержать модификатор async, и иметь один или более вызовов await. Такой метод при вызове вернет экземпляр запущенной задачи (Task).

**Task.Run(Action)** - запускает и возвращает запущенную задачу  
**Task.Factory** - фабрика для создания и конфигурации задач

**Task.FormResult()** - создает завершенную задачу по результату
**Task.Delay()** - асинхронно ждет указанный промежуток времени

**Task.WaitAll(param tasks)** - ждет выполнения всех переданных задач
**Task.WaitAny(param tasks)** - ждет завершение одной задачи

**Task.WhenAll()** - WaitAll, но создает задачу с массивом результатов
**Task.WhenAny()** - WaitAny, но создает задачу результат которой первая задача

**TaskInst.Start()** - ставит объект задачи на выполнение.
**TaskInst.Wait()** - блокирует вызывающий поток до завершения задачи
**TaskInst.Result** - блокирует вызывающий поток и возвращает результат задачи

**TaskInst.ContinueWith(Action\<Task>)** - регистрирует задачу на выполнение после
**TaskInst.RunSynchronously()** - запускает задачу синхронно в текущем потоке

---

Параллельное выполнение задач может занимать много времени. И иногда может возникнуть необходимость прервать выполняемую задачу. Для этого платформа .NET предоставляет механизм структуру CancellationToken.

**CancellationTokenSource** - позволяет создавать специальные токены, который содержат информацию про состояние  отмены. И позволяет отменить операцию Cancel(). Является Unmanaged ресурсом и требует вызова Dispose.

**CancellationToken** - собственно сам токен, у него есть свойство IsCancellationRequested, которое показывает состояние отмены и также метод ThrowIfCancellationRequested(), который выбрасывает исключение (OperationCanceledException) при отмене.

Большинство методов стандартной библиотеки уже имеют перегрузку, которая принимает токен. Чтобы его создать, нужно сначала создать фабрику CTS. Дальше с ее помощью сгенерировать токен и передать его в качестве аргумента в наш асинхронный метод или напрямую в задачу через одну из ее перегрузок.

Чтобы отменить задачу нужно вызвать метод Cancel у экземпляра CTS. Он переведет все выпущенные токены в отмененное состояние. Внутри асинхронного метода сработает проверка токена и работа будет завершена.

Также CancellationTokenSource позволяет установить таймаут по истечению которого автоматически отменяться все токены. Для этого вызывается cts.CancelAfter(time)

---
## Асинхронный метод возвращает:

1. **Void** - просто запуск задачи без возможности отслеживания
    
2. **Task** - дает возможность ожидать задачу и отслеживать ее состояние
    
3. **Task\<T>** - после await вернет указанный тип T
    
4. **ValueTask\<T>** - возвращает значение без аллокации память

## Свойства и методы Task:

1. **AsyncState** - возвращает объект состояния задачи
    
3. **Id** - возвращает идентификатор текущей задачи
    
4. **Status** - возвращает статус задачи.
    
5. **IsCompleted** - возвращает true, если задача завершена
    
6. **IsCanceled** - возвращает true, если задача была отменена
    
7. **IsFaulted** - true, если задача завершилась с исключением
    
8. **IsCompletedSuccessfully** - true, если задача завершилась успешно
    
9. **Exception** - возвращает исключения, возникшего при выполнении задачи

[[🟡ExecutionModel]]