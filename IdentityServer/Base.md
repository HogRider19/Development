**IdentityServer4** - это платформа OIDC и OAuth с открытым исходным кодом. Другими словами, это поставщик аутентификации для ваших решений.

Основная идея - централизовать поставщика аутентификации. Несколько связанных сервисов могут иметь централизованную систему аутентификации, а не определять свою логику входа. С помощью IdentityServer вы можете централизовать контроль доступа, чтобы каждый API был защищен центральным IdentityServer.

**OAuth 2.0** - это протокол и стандарт авторизации. Он позволяет приложениям получить доступ к защищенным ресурсам, например к Web API.

**OIDC** - это протокол и стандарт аутентификации, он не дает доступ к ресурсам, но т.к. он разработан поверх протокола авторизации OAuth 2.0, он позволяет получить параметры профиля пользователя как будто вы получили доступ к ресурсу UserInfo.

---

### Концепция IdentityServer

Идея довольно проста и понятна. Пользователи используют клиент для доступа к данным. Как только пользователи проходят аутентификацию для использования клиента, клиент получает возможность отправлять запросы к защищенному API. 

Помните, что ресурсы клиента и API защищены одним объектом - IdentityServer. Клиент запрашивает токен доступа, с помощью которого он может получить доступ к ответам API. Таким образом, мы централизуем механизм аутентификации на одном сервере.

---

### Обязанности IdentityServer

- Защитите свои ресурсы
- Аутентифицировать пользователей 
- Обеспечить управление сеансами и единый вход
- Управлять и аутентифицировать клиентов
- Выдавать токены идентификации и доступа клиентам
- Проверять токены

---

1) Клиент запрашивает у пользователя разрешение пройти авторизацию от его имени. Клиент - это клиентское приложение обращающиеся к защищённым ресурсам от имени владельца ресурсов. Ресурс - это все наши защищенные сервисы Web API.

2) User разрешает клиентскому приложению пройти авторизацию от его имени, например, вводит логин и пароль. Логин и пароль будут являться грантом авторизации для клиентского приложения. User (владелец ресурса) - программа или человек, который может выдать доступ к защищённым ресурсам, например, путем ввода логина (username) и пароля (password);

3) Клиентское приложение запрашивает токен доступа у IdentityServer4 путём предоставления информации о самом себе (client_id, client_secret), предоставления разрешения на авторизацию от пользователя (username, password) и предоставления grant_type и scope. Затем сервер авторизации проверяет подлинность клиента и реквизиты владельца ресурса (логин и пароль).
 
 >[!info] Про Client и IdentityServer
 >Протокол OAuth 2.0 проводит аутентификации не только пользователя, но и клиентского приложения, осуществляющего доступ к ресурсам. Для этого протокол предусматривает такие параметры как client_id и client_secret. 
 >
 >**сlient_id** - это идентификатор клиентского приложения, используемый IdentityServer4 для поиска информации о клиенте.
 >**client_secret** - аналог пароля для клиентского приложения и используется для аутентификации клиентского приложения на IdentityServer4.
 >**grant_type** — тип гранта или тип разрешения на авторизацию. Тип разрешения на авторизацию зависит от используемого приложением метода запроса авторизации
 >
 >client_secret клиента должен быть известен только приложению и API. Исходя из вышесказанного делаем вывод что IdentityServer4 должен знать о своих клиентах.

4) Если подлинность приложения подтверждена и разрешение на авторизацию действительно, IdentiryServer4 создаёт access-токен (токен доступа) для приложения и необязательный ключ обновления (refresh-токен). Процесс авторизации завершён. Если запрос недопустимый или несанкционированный, то сервер авторизации возвращает код с соответствующим сообщением об ошибке.

5) Клиентское приложение обращается за данными к защищенному Web API, предоставляя при этом токен доступа для авторизации. Если код ответа сервера ресурсов 401, 403 или 498, то токен доступа, используемый для аутентификации, недействителен или просрочен.

6) Если токен действителен, Web API предоставляет данные приложению.

![[IdentityServer.png]]

### Токены:

- **identity-token** - результат процесса аутентификации. Содержит идентификатор пользователя и информации о том, как и когда пользователь проходит аутентификацию. Можно расширить своими данными.
- **access-token** - передается защищенному API и используется им для авторизации (разрешения доступа) к своим данным.
- **refresh-token** - необязательный параметр, который сервер авторизации может возвратить в ответ на запрос токена доступа.

**Authenticatation Server Url** - конечная точка для получения ключа доступа. Все запросы на предоставление и возобновление ключей доступа будем направлять на этот URL.

**Resource Url** - URL-адрес защищенного ресурса, по которому нужно обращаться, чтобы получить доступ к нему, передавая ему ключ доступа в заголовке авторизации.

**Scope** — это необязательный параметр. Он определяет область видимости. Токен доступа, возвращенный сервером, будет обеспечивать доступ только к тем службам, которые входят в эту область.

---

Протокол OAuth 2.0 определяет следующие типы грантов:

1) **Authorization code** - Является одним из наиболее распространённых типов разрешения на авторизацию, т.к. хорошо подходит для серверных приложений, где исходный код приложения и секрет клиента не доступны посторонним
 
2) **Implicit** - неявный тип разрешения на авторизацию используется мобильными и веб-приложениями, где конфиденциальность секрета клиента не гарантирована

3) **Resource owner** - этот тип разрешения стоит использовать только в том случае, когда клиентское приложение пользуется доверием пользователя и пользователь спокойно относится к вводу своего логина и пароля. Этот тип разрешения должен использоваться только в том случае, когда другие варианты не доступны.

5) **Client data** - используются при доступе приложения к API. Это может быть полезно, например, когда приложение хочет обновить собственную регистрационную информацию на сервисе или URI перенаправления.