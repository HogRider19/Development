Fetch API предоставляет упрощенный и в тоже время гибкий и мощный инструмент для обращения к сетевым ресурсам по сравнению со стандартным XMLHttpRequest.

Ключевым элементом этого Fetch API является функция fetch(). Она реализована в различных контекстах. В частности, в браузере она реализована интерфейсом Windows.

---
#### Fetch

`fetch(resource: str, init: obj) -> Promise` - сигнатура функции fatch

`resource` - строка с адресом ресурса к которому функция будет обращаться.
`init` - объект с дополнительными настройками запроса.

В качестве результата асинхронной операции fetch возвращает объект response, который инкапсулирует в себе информацию о полученном ответе.

---
#### Init

Функция fetch() может дополнительно принимать опции запроса в виде второго необязательного параметра. Параметр `init` представляет сложный объект, который может имеет большой набор опций:

- **method** - метод запроса, например, GET, POST, PUT и т.д.
    
- **headers** - устанавливаемые в запросе заголовки в виде словаря
    
- **body** - тело запроса данные, которые добавляются в запрос.
    
- **mode** - режим запроса, например, `cors`, `no-cors` и `same-origin`
    
- **redirect** - устанавливает, как реагировать на редиректы. Может принимать следующие значения:  `follow`: автоматически применять редирект, `error`: при редиректе генерировать ошибку.

---
#### Response

Для представления ответа от сервера в Fetch API применяется интерфейс Response. Функция fetch() возвращает объект Promise, resolve в котором в качестве параметра получает объект Response с полученным от сервера ответом.

**body** - содержимое ответа в виде объекта ReadableStream
**bodyUsed** - хранит bool, которое указывает, было ли body уже использовано.
**headers** - набор заголовков ответа в виде объекта Headers

**status** - хранит статусный код ответа
**ok** - возвращает true при возвращении успешного статус кода
**redirected** - возвращает true, если в процессе были переадресации
**statusText** - хранит сообщение статуса, которое соответствует статусному коду

**type** - хранит тип ответа
**url** - хранит адрес URL. При переадресации, хранит последний путь


**json()** - получает тело ответа в виде JSON
**text()** - получает тело ответа в виде текста

**arrayBuffer()** - получает тело ответа в виде ArrayBuffer (Бинарные данные)
**blob()** - получает тело ответа в виде Blob (Бинарные данные)
**formData()** - получает тело ответа в виде FormData (Работа с формами)

**redirect()** - возвращает новый объект Response с другим адресом URL
**clone()** - возвращает копию текущего объекта Response
**error()** - возвращает новый Response, который связан с возникшей ошибкой сети

---
#### Headers

С помощью свойства headers можно получить заголовки ответа, которые представляют интерфейс Headers. Для получения данных из заголовков мы можем воспользоваться один из следующих методов интерфейса Headers:

**entries()** - возвращает итератор, который позволяет пройтись по всем заголовкам
**forEach()** - осуществляет перебор заголовков
**get()** - возвращает значение определенного заголовка
**has()** - возвращает true, если установлен определенный заголовок
**keys()** - получает все названия установленных заголовков
**values()** - получает все значения установленных заголовков