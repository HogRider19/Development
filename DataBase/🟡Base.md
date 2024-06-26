На заре развития вычислительной техники обрабатываемые данные являлись частью программ: они располагались сразу с кодом программы в так называемом сегменте данных. Следующим шагом стало хранение данных в отдельных файлах.

Недостатком этих двух подходов являлась зависимость программ от данных: сведения о структуре данных включались в код программы. При изменении структуры данных необходимо было вносить изменения в логику поведения программы.

Логичным продолжением этой эволюции является перенос описания данных в сам массив данных. Это позволило обеспечить независимость данных от программ.

>[!info] Метаданные
> Основным принципом организации баз данных является совместное хранение данных и их метаданных, которые описывают структуру хранения информации.

**Инфологическая модель** – это описание предметной области, выполненное без ориентации на используемые в дальнейшем программные и технические средства. Цель инфологического проектирования в определении семантики предметной области.

**Даталогическая модель** - это описание самих данных и связи между данными. Для представления даталогической модели БД используется ER-диаграмма, на которой изображаются сущности проектируемой БД и связи, устанавливаемые между ними.

**База данных** - это совокупность данных, организованных по определённым правилам, которые предусматривают общие принципы управления хранимыми данными, независимо от используемых в последующем прикладных программ.

**Система управления базой данных** - это совокупность программ и языковых средств, предназначенных для управления данными в базе данных, ведения базы данных и обеспечения взаимодействия её с используемыми прикладными программами.

**Structured Query Language (SQL)** – это язык программирования для управления и обработки информации в базе данных на основе реляционной модели.

Решение SQL было изобретено в 1970-х годах на основе реляционной модели данных. Изначально был известен как структурированный английский язык запросов (SEQUEL). Позднее этот термин был сокращен до SQL из-за юридических причин с названием.

Язык SQL катализируется на разные виды запросов в зависимости от исполняемых запросов действия. Можно выделить основные категории запросов:

**DDL (Data Definition Language)**. Это язык определения данных, с помощью которого создается база данных и дается описание ее структуры. DDL запрос позволяет настроить правила размещения различной информации в таблице базы данных.

**DML (Data Manipulation Language)**. Это язык работы с данными. Как правило, применяемые команды нужны для внесения изменений в уже существующие данные, их удаления и сохранения, обновления записей и т.д.

**DCL (Data Control Language)**. Это язык управления данными. К ним относятся команды, при помощи которых могут быть внесены изменения в настройки расширений, доступа и другие элементы системы управления базой данных.

**TCL (Transaction Control Language)**. Это язык управления транзакциями. TCL запрос используется, когда необходимо объединить несколько команд DML, направленных на изменение данных, в наборы атомарных операций.

---
PostgreSQL является объектно-реляционной системой управления базами данных, наиболее развитая из открытых СУБД в мире. Имеет открытый исходный код и является альтернативой коммерческим базам данных.

PostgreSQL называют бесплатным аналогом Oracle Database. Обе системы адаптированы под большие проекты и высокую нагрузку. 

Но есть разница: они по-разному хранят данные, предоставляют разные инструменты и различаются возможностями. Важная особенность PostgreSQL в том, что эта система feature-rich: так называют проекты с широким функционалом.

Говоря техническим языком, PostgreSQL реализован в архитектуре клиент-сервер. Рабочий сеанс PostgreSQL включает в себя главный процесс `postgres`, управляющий файлами баз данных, принимающий подключения и выполняющий запросы.

Сервер PostgreSQL может обслуживать одновременно несколько подключений клиентов. Для этого он порождает отдельный процесс для каждого подключения. Клиент и серверный процесс общаются, не затрагивая главный процесс `postgres`.
