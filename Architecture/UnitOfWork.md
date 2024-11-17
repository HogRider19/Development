**Unit of Work** - это паттерн определяющий логическую транзакцию т.е. атомарную синхронизацию изменений в объектах, помещённых в объект UoW с хранилищем.

Если обратиться к исходному описанию этого паттерна у Мартина Фаулера - то видно что объект, реализующий этот паттерн отвечает за накопление информации о том какие объекты входят в транзакцию и каковы их изменения относительно исходных значений в хранилище.

UnitOfWork представляет собой класс, реализующий интерфейс IDisposable и содержащий метод Save. Также он имеет набор свойств для доступа к хранимым репозиториям доступа к данным.