**HyperText Transfer Protocol(HTTP)** — широко распространённый протокол передачи данных, изначально предназначенный для передачи гипертекстовых документов.

Предполагает использование клиент-серверной структуры передачи данных. Клиентское приложение формирует запрос и отправляет его на сервер, после чего сервер обрабатывает данный запрос, формирует ответ и передаёт его обратно клиенту.

Его часто используется как протокол передачи информации для других протоколов прикладного уровня, таких как SOAP, RPC и WebDAV. В таком случае говорят, что протокол HTTP используется как транспорт.

Обычно передача данных по протоколу HTTP осуществляется через TCP соединение. Серверное программное обеспечение при этом обычно использует порт 80.

## Методы запроса:

- **GET** - запрашивает представление ресурса. 
- **HEAD** - запрашивает ресурс так же, как и метод GET, но без тела ответа. 
- **POST** - используется для отправки сущностей к определённому ресурсу. 
- **PUT** - заменяет все текущие представления ресурса данными запроса. 
- **PATCH** - используется для частичного изменения ресурса 
- **DELETE** - удаляет указанный ресурс. 
- **CONNECT** - устанавливает "туннель" к серверу, определённому по ресурсу. 
- **OPTIONS** - используется для описания параметров соединения с ресурсом.