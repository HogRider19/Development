**System.Collections.Generic** - пространство имен коллекций

---

**List<Т>** - динамический массив

lst.Count - длинна списка  
lst.Capacity - емкость списка  
lst.Add(item) - добавляет элемент в конец списка  
lst.AddRange(Inum) - добавляет список или массив в конец  
lst.BinarySearch(item) - бинарный поиск  
lst.CopyTo(array) - копирует список в массив  
lst.Contains(item) - true если элемент есть в списке  
lst.Clear() - очищает список  
lst.Exists(Predicate\<T> match) - true, если есть элемент паттерна  
Find(Predicate\<T> match) - ищет элемент по паттерну  
lst.Insert(index, item) - вставляет элемент в список по индексу  
lst.Remove(item) - удаляет первый элемент item из списка  
lst.RemoveAt(index) - удаление элемента по указанному индексу 
lst.Reverse() - изменяет порядок элементов  
lst.Sort() - сортирует список

---

**LinkedList\<T>** - двусвязный список

AddAfter(item, new) - вставляет узел new в список после узла item  
AddBefore(item, T value) - вставляет узел new в список перед узла item  
AddFirst(value) - вставляет новый узел value в начало списка  
AddLast(value) - вставляет новый узел value в конец списка  
RemoveFirst() - удаляет первый узел из списка  
RemoveLast() - удаляет последний узел из списка

---

**Queue\<T>** - очередь

Clear() - очищает очередь  
Contains(item) - true, если элемент item имеется в очереди  
Dequeue() - извлекает и возвращает первый элемент очереди  
Enqueue(item) - добавляет элемент в конец очереди  
Peek() - возвращает первый элемент без его удаления

---

**Stack\<T>** - стек

Clear() - очищает стек  
Contains(item) - true если элемент есть в стеке  
Push(item) - добавляет элемент в стек в верхушку стека  
Pop() - извлекает и возвращает первый элемент из стека  
Peek() - просто возвращает первый элемент из стека без его удаления

---

**Dictionary<K, V>** - ассоциативный массив

Add(key, value) - добавляет новый элемент в словарь  
Clear() - очищает словарь  
ContainsKey(K key) - true если есть элемент с ключом key  
ContainsValue(value) - true если есть элемент с значением value  
Remove(key) - удаляет по ключу элемент из словаря  
TryGetValue(key, out value) - безопасное получение по ключу  
TryAdd(K key, V value) - безопасное добавление элемента

---

**public IEnumerator GetEnumerator()** - реализация интерфейса IEnumerable

**public bool MoveNext()**------|  
**public object Current()**-------| - реализация IEnumerator  
**public void Reset()**-----------|

[[🟡ExecutionModel]]