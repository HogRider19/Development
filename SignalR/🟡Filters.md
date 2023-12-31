Фильтры представляют специальные компоненты, которые выполняют до и после выполнения Hub. Фильтры могут быть полезны, если необходимо перед и/или после выполнения Hub определить некоторую программную логику

Для регистрации фильтра применяется перегрузка метода AddSignalR(), который в качестве параметра принимает делегат. Параметр этого делегата - объект HubOptions с помощью метода AddFilter() добавляет фильтр в качестве сервиса. Для добавления фильтра метод AddFilter типизируется типом фильтра.

Чтобы задействовать фильтр, его надо зарегистрировать. Это можно сделать глобально - для всех хабов, либо локально - для определенного хаба.

Фильтры внедряются в виде сервисов в приложение. Однако в зависимости от типа регистрации сервис будет иметь определенный тип жизненного цикла. Например, при регистрации в виде типизации метода AddFilter при каждом отдельном обращении к Hub будет создаваться новый объект фильтра

Если же в метод AddFilter передается объект фильтра, то будет создаваться один фильтр, который будет существовать на протяжении всей жизни приложения, то есть это будет signleton-сервис

Функционально фильтры представляют классы, которые реализуют интерфейс IHubFilter. Данный интерфейс определяет три метода:

---

### InvokeMethodAsync

Вызывается при каждом обращении к методу Hub.

Первый параметр - объект HubInvocationContext представляет контекст вызова Hub и хранит всю связанную информацию в виде свойств:

**Context** - контекст хаба - объект HubCallerContext 
**Hub** - вызываемый хаб в виде объекта класса Hub 
**HubMethod** - вызываемый метод хаба - объект MethodName
**HubMethodName** - название вызываемого метода Hub 
**HubMethodArguments** - аргументы вызываемого метода Hub
**ServiceProvider** - провайдер сервисов - объект IServiceProvider

Второй параметр - next представляет следующий в конвейере фильтр Hub. Если в конвейере больше нет фильтров, то представляет вызов метода Hub. Метод возвращает результат работы метода Hub.

---

### OnConnectedAsync

Вызывается при обращении к методу OnConnectedAsync.

Первый параметр - объект HubLifetimeContext представляет контекст вызова метода OnConnectedAsync и имеет следующие свойства:

**Context** - контекст Hub объект HubCallerContext 
**Hub** - вызываемый Hub в виде объекта класса Hub
**ServiceProvider** - провайдер сервисов - объект IServiceProvider

Второй параметр - next представляет следующий в конвейере фильтр хаба. Если в конвейере больше нет фильтров, то представляет вызов метода хаба.

---

### OnDisconnectedAsync

Вызывается при обращении к методу OnDisconnectedAsync хаба.

Первый параметр - объект HubInvocationContext представляет контекст вызова метода OnDisconnectedAsync

Второй параметр - Exception представляет возможную ошибку, которая может произойти при отключении.

Третий параметр - next представляет следующий в конвейере фильтр хаба. Если в конвейере больше нет фильтров, то представляет вызов метода хаба.