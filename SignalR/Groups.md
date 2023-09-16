Группы в SignalR представляют коллекцию подключений, которые ассоциированы с именем группы. SignalR позволяет добавлять и удалять клиентов из группы. Один клиент может входить сразу в несколько групп.

**Hub.Goups** - свойство IGroupManager для работы с группами в Hub

**Goups.AddToGroupAsync(connectionId, groupName)** - добавляет клиента с идентификатором connectionId в группу groupName **Goups.RemoveFromGroupAsync(connectionId, groupName)** - удаляет клиента с идентификатором connectionId из группы groupName

---

### Методы IHubCallerClients для работы с группами

**Group(string groupName)** - вызывает метод у клиентов группы

**Groups(groupNames)** - вызывает метод у клиентов групп

**OthersInGroup(OthersInGroup)** - вызывает метод у клиентов определенной группы за исключением текущего клиента

**GroupExcept(groupName, connectionIds)** - вызывает метод у клиентов группы по имени groupName за исключением тех клиентов, id которых передаются в качестве второго параметра