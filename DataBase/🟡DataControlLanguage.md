DCL (Data Control Language) представляет язык управления данными. К ним относятся команды, при помощи которых могут быть внесены изменения в настройки расширений, доступа и другие элементы системы управления базой данных.

PostgreSQL использует концепцию ролей для управления разрешениями на доступ к базе данных. Роль можно рассматривать как пользователя базы данных или как группу пользователей, в зависимости от того, как роль настроена.

Роли могут владеть объектами базы данных например, таблицами и функциями и выдавать другим ролям разрешения на доступ к этим объектам, управляя тем, кто имеет доступ и к каким объектам. Кроме того, можно предоставить одной роли членство в другой роли, тогда она может использовать права других ролей.

Концепция ролей включает в себя концепцию пользователей  и групп. До версии 8.1 в PostgreSQL пользователи и группы были отдельными сущностями, но теперь есть только роли. Любая роль может использоваться в качестве пользователя и группы.

---

Роли базы данных концептуально полностью отличаются от пользователей операционной системы. На практике поддержание соответствия между ними может быть удобным, но не является обязательным. Роли базы данных являются глобальными для всей установки кластера базы данных. Для создания  используется `CREATE ROLE`.

Система хранит специальные системные каталоги, в которых описывается информация о ролях. Для получения списка существующих ролей, обратитесь к таблице `pg_roles`.

```sql
CREATE ROLE role_name [attrs];   // Создание роли
DROP ROLE role_name;             // Удаление роли 

SELECT rolname FROM pg_roles;    // Чтение имен всех ролей кластера
```

Роль базы данных может иметь атрибуты, определяющие её полномочия и взаимодействие с системой аутентификации клиентов приложения. Для присвоению определенной роли советующих атрибутов, их нужно перечислить при создании роли.

**==LOGIN==**. Только роли с атрибутом `LOGIN` могут использоваться для начального подключения к базе данных. Роль с атрибутом `LOGIN` можно рассматривать как пользователя базы данных. `CREATE USER` автоматически добавляет к роли `LOGIN`.

**==SUPERUSER==**. Суперпользователь базы данных обходит все проверки прав доступа, за исключением права на вход в систему. Это опасная привилегия и она не должна использоваться небрежно. Лучше всего выполнять большую часть работы не как суперпользователь. Выдавать такие права может только другой суперпользователь.

**==CREATEDB==**. Роль должна явно иметь разрешение на создание базы данных за исключением суперпользователей, которые пропускают все проверки.

**==CREATEROLE==**. Роль должна явно иметь разрешение на создание других ролей. Такие роли могут создавать, изменять и удалять роли, которые были им созданы. Однако им нельзя ни создавать, ни каким-либо образом влиять на роли `SUPERUSER`.

**==REPLICATION==**. Роль должна иметь явное разрешение на запуск потоковой репликации. Роль, используемая для потоковой репликации, также должна иметь атрибут `LOGIN`.

**==PASSWORD==**. Пароль имеет значение, если метод аутентификации клиентов требует, чтобы пользователи предоставляли пароль при подключении к базе данных. Методы аутентификации `password` и `md5` используют пароли. База данных и операционная система используют раздельные пароли. Пароль указывается при создании роли.

**==NOINHERIT==**. По умолчанию роль наследует права ролей, членом которых она является. Однако можно создать роль, которая не будет наследовать права по умолчанию. 

**==BYPASSRLS==**. Разрешение обходить все политики защиты на уровне строк (RLS) нужно давать роли явно. Право выдать такой атрибут имеет только суперпользователь.

---

Часто бывает удобно сгруппировать пользователей для упрощения управления правами: права можно выдавать для всей группы и у всей группы забирать. Для этого создаётся роль, представляющая группу, а затем членство в ней выдаётся другим ролям.

Для настройки групповой роли сначала нужно создать саму роль. Обычно групповая роль не имеет атрибута `LOGIN`, хотя при желании его можно установить. Затем в неё можно добавлять или удалять членов, используя команды `GRANT` и `REVOKE`.

```sql
GRANT goup_role TO role_1, ..., role_n;
REVOKE goup_role FROM role_1, ..., role_n;
```

Членом роли может быть и другая групповая роль, так как в действительности между ними нет различий. База данных не допускает замыкания членства по кругу. 

Члены групповой роли могут использовать её права двумя способами. Во-первых, члены группы, которым было предоставлено членство с атрибутом `SET`, могут выполнить команду `SET ROLE`, чтобы временно стать групповой ролью.

Во-вторых, члены группы, которым было предоставлено членство с атрибутом`INHERIT`, автоматически используют права групповой роли, в том числе и унаследованные ею.

```sql
CREATE ROLE admin SUPERUSER;

GRANT admin TO role_1 WITH INHERIT TRUE; 
GRANT admin TO role_2 SET TRUE;           
```

Атрибуты роли `LOGIN`, `SUPERUSER`, `CREATEDB` и `CREATEROLE` являются особыми правами, они никогда не наследуются как обычные права на объекты базы данных. По умолчанию разрешения на переключение и наследование включены для всех членов.

Чтобы ими воспользоваться, необходимо переключиться на роль, имеющую этот атрибут, с помощью команды `SET ROLE`. Поэтому в предыдущем примере первая роль не получит права супер пользователя без переключения на нее. 

---

Так как роли могут владеть объектами баз данных и иметь права доступа к объектам других, удаление роли не сводится к немедленному действию `DROP ROLE`. Сначала должны быть удалены и переданы другим владельцами все объекты. Владение объектами можно передавать в индивидуальном порядке, применяя `ALTER`.

```sql
ALTER TABLE table_name OWNER TO new_owner;
```

Кроме того, для переназначения какой-либо другой роли владения сразу всеми объектами, принадлежащими удаляемой роли, можно применить `REASSIGN OWNED`. Ввиду того что данная команда  не может обращаться к объектам в других базах данных, её необходимо выполнить в каждой базе, которая содержит объекты.

```sql
REASSIGN OWNED BY old_owner TO new_owner;
DELETE ROLE old_owner;
```

---

Функции, триггеры и политики защиты на уровне строк позволяют пользователям внедрять код в обслуживающие процессы, который может быть непреднамеренно выполнен другими пользователями. Таким образом эти механизмы позволяют пользователям запускать троянский код относительно просто.

Лучшая защита от этого - строгое ограничение круга лиц, которые могут создавать объекты. Там где это невозможно, пишите запросы так, чтобы они ссылались только на объекты владельцам которых вы точно доверяете управление .

Функции выполняются внутри серверного процесса с полномочиями пользователя операционной системы, запускающего сервер базы данных. Если используемый для функций язык программирования разрешает неконтролируемый доступ к памяти, то это даёт возможность изменить внутренние структуры данных сервера.

Таким образом, помимо всего прочего, такие функции могут обойти ограничения доступа к системе. Языки программирования, допускающие такой доступ являются ненадежными и создавать на них функции разрешено только суперпользователям.

---

Когда в базе данных создаётся объект, ему назначается владелец. Владельцем обычно становится роль, с которой был выполнен оператор создания. Для большинства типов объектов в исходном состоянии только владелец или суперпользователь может делать с объектом всё, что угодно. Для иного поведения нужно выдавать права другим ролям.

Существует несколько типов прав: `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `REFERENCES`, `TRIGGER`, `CREATE`, `CONNECT`, `TEMPORARY`, `EXECUTE`, `USAGE`, `SET`, `ALTER SYSTEM`. Набор прав, применимых к определённому объекту, зависит от типа объекта.

Право изменять или удалять объект является неотъемлемым правом владельца объекта, его нельзя лишиться или передать другому. Однако как и все другие права, это право наследуют члены роли-владельца.

Для назначения прав применяется команда `GRANT`. Которая позволяет назначить вышеперечисленные права определенному пользователю для объекта базы данных. Чтобы лишить пользователей ранее данных им прав, используйте команду `REVOKE`.
Если вместо конкретного права написать `ALL`, роль получит все права. 

Для назначения права всем ролям в системе можно использовать специальное имя роли: `PUBLIC`. Также для упрощения управления ролями, когда в базе данных есть множество пользователей, можно настроить групповые роли.

```sql
GRANT rool_type ON object_name TO role;
REVOKE ALL ON object_name FROM PUBLIC;
```

Владелец объекта может лишить себя обычных прав, например запретить не только всем остальным, но и себе, вносить изменения в таблицу. Однако владельцы всегда имеют возможность управлять правами, так что они могут в любой момент их вернуть.

**==SELECT==**. Позволяет выполнять запрос для любого столбца или перечисленных столбцов в заданной таблице, представлении, матпредставлении или другом объекте табличного вида. Также позволяет выполнять `COPY TO`. Помимо этого, данное право требуется для обращения к существующим значениям столбцов в `UPDATE`, `DELETE` или `MERGE`.

**==INSERT==**. Позволяет вставлять с помощью `INSERT` строки в заданную таблицу, представление и тому подобное. Может назначаться для отдельных столбцов. В этом случае только этим столбцам можно присваивать значения в команде `INSERT`, другие столбцы записи  получат значения по умолчанию.

**==UPDATE==**. Позволяет изменять с помощью `UPDATE` данные во всех, либо только перечисленных, столбцах в заданной таблице, представлении и тому подобное.

**==DELETE==**. Позволяет удалять с помощью команды `DELETE` строки из таблицы, представления и тому подобное в выбранной базе данных.

**==TRUNCATE==**. Дает выбранному пользователю права на использование команды  `TRUNCATE`, которая в свою очередь полностью опустошает таблицу.

**==REFERENCES==**. Позволяет создавать ограничение внешнего ключа, обращающееся к таблице или определённым столбцам таблицы.

**==TRIGGER==**. Позволяет создавать триггер для таблицы, представления и подобным им сущностям в выбранной базе данных.

**==CREATE==**. Для баз данных это право позволяет создавать схемы и публикации. Для схем это право позволяет создавать новые объекты в заданной схеме. Для табличных пространств это право позволяет создавать таблицы, индексы и временные файлы в определённом табличном пространстве, а также создавать в нем базы данных.

**==CONNECT==**. Позволяет подключаться к базе данных. Это право проверяется при установлении соединения в дополнение к условиям, определённым в `pg_hba.conf`.

**==TEMPORARY==**. Позволяет указанному пользователю базы данных создавать и удалять временные таблицы в определённой базе данных.

**==EXECUTE==**. Позволяет вызывать функцию или процедуру, в том числе использовать любые операторы, реализованные данной функцией. Это единственный тип прав, применимый к функциям и процедурам.

**==SET==**. Позволяет задать для параметра конфигурации сервера новое значение в текущем сеансе. Хотя это право может быть предоставлено для любого параметра, оно имеет смысл только для тех параметров, для которых требуются права суперпользователя.

---

В дополнение к стандартной системе прав SQL, на уровне таблиц можно определить политики защиты строк, ограничивающие для пользователей наборы строк.

Это называется также защитой на уровне строк RLS (Row-Level Security). По умолчанию таблицы не имеют политик, так что если система прав SQL разрешает пользователю доступ к таблице, все строки в ней одинаково доступны для чтения или изменения.

Когда для таблицы включается защита строк `ALTER TABLE ENABLE ROW LEVEL SECURITY`, все обычные запросы к таблице на выборку или модификацию строк должны разрешаться политикой защиты строк. Однако на владельца они не действуют. 

Если политика для таблицы не определена, применяется политика запрета по умолчанию, так что никакие строки в этой таблице нельзя увидеть или изменить.

Политики защиты строк могут применяться к определённым командам или ролям. Политику можно определить как применяемую к командам `ALL`, `SELECT`, `INSERT`, `UPDATE` и `DELETE`. Также политику можно связать с несколькими ролями.

Чтобы определить, какие строки будут видимыми или могут изменяться в таблице, для политики задаётся выражение, возвращающее логический результат. Это выражение будет вычисляться для каждой строки перед другими условиями или функциями.

Для создания политик предназначена команда `CREATE POLICY`, для изменения `ALTER POLICY`, а для удаления - `DROP POLICY`. Чтобы включить или отключить защиту строк для определённой таблицы, воспользуйтесь командой `ALTER TABLE`.

Каждой политике назначается имя, при этом для одной таблицы можно определить несколько политик. Так как политики привязаны к таблицам, каждая политика для таблицы должна иметь уникальное имя. В разных таблицах могут иметь одинаковые.

```sql
CREATE TABLE table_name (col_1, col_2, col_3);
ALTER TABLE table_name ENABLE ROW LEVEL SECURITY;
CREATE POLICY pol_name ON table_name TO role USING (cond) WITH CHECK (cond);
```

[[DataBase/🟡Base|🟡Base]]