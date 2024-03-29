
https://edu.postgrespro.ru/dev1-12/dev1_02_arch_general.html

### Кластер БД
При запуске postgres создается instance, который поднимает кластер баз данных, необходимых для хранения как пользовательских данных, так и системных данных.

### Протокол
Для общения с кластером бд используется протокол (https://postgrespro.ru/docs/postgresql/12/protocol) или же драйвер, который реализует протокол. Клиент связывается с сервером, аутентифицируется и посылает ему запросы.

### ACID
Одна из особенностей реляционных бд - наличие транзакций - свойство, которое позволяет бд сохранять целостность, а также согласованность системы. Транзакция обязана соответствовать следующим 4 требованиям:
1)  А (Atomicity) - Атомарность, транзакции должны выполнятся либо все запросы, либо никакие, для этого существуют специальные ключи: **BEGIN** - для начала транзакции, **COMMIT** - для фиксации изменений, **ROLLBACK** - для отмены всех изменений
2) C (Consistency) - Согласованность, до начала транзакции и после окончания транзакции система должна быть в согласованном виде - соответствовать ограничениям, структуре  и тд.
3) I (Isolation) - Изоляция, транзакции не должны влиять друг на друга работая одновременно.
4) D (Durability) - Долговечность, в случае внезапного отключения системы, данные должны быть сохранены если они зафиксированы. 

### Выполнение запросов
После того как запрос отправляется на сервер в виде текста, он проходит различные синтаксические и семантические проверки и разборы, в случае успеха разобранный запрос в виде дерева передается планировщику, который основываясь на данных статистики, а также запросе выбирает тот или иной путь для выполнения команд запроса, например использовать тот или иной индекс.

Для того чтобы сократить время разбора запроса можно заранее подготовить(разобрать) запрос для последующего его выполнения, с помощью команды **PREPARE** (https://postgrespro.ru/docs/postgresql/12/sql-prepare).

### Курсоры
Одна из дополнительных техник реализаций пагинации может служить использование курсоров в бд. Курсор позволяет открыть "окно" между клиентом и сервером и получать данные постепенно, при этом есть возможность получать данные не построчно, а сразу n-количеством записей. Определяется курсор благодаря ключевому слову DECLARE (https://postgrespro.ru/docs/postgresql/12/sql-declare). Данный способ пагинации отлично подходит для новостной ленты, где нет страниц и пользователь просто листает новости вниз. Пагинация с помощью курсора является более производительнее нежели использование limit/offset, но при этом пагинация с курсором очень не эффективна для пагинации со страницами, так как нельзя задать offset. Подробнее - https://ignaciochiazzo.medium.com/paginating-requests-in-apis-d4883d4c1c4c

### Процессы и память
При создании сервера postgres создает главный процесс, который называется postmaster, он контролирует работу всех остальных узлов. Из создаваемых узлов postmaster'ом можно перечислить: Процессы, обрабатывающие запросы клиентов (backend), фоновые процессы. Каждый из подпроцессов имеет локальную память, где хранит ту или иную память. Например в локальной памяти backend процессов хранятся разобранные запросы, их планы, состояние курсоров, кеш системного каталога. Так же postmaster выделяет общую память для того, чтобы подпроцессы могли обмениваться информацией между собой.
![[Pasted image 20231221222845.png|center]]


При подключении клиента к серверу, для него создается отдельный процесс backend, при маленьком количестве соединений это работает хорошо, но при большом количестве появляется необходимость в использовании пула соединений, например PgBouncer, который будет держать в себе клиентов и направлять их к одному backend подпроцессору.

Так как подключений много, то и соблюдать транзакции становится сложно. Postgres решает данную проблему с помощью механизма MVCC, или же мультиверсионности. Благодаря данному механизму данные могут одновременно существовать в разных версиях, но при этом оставаться согласованными. Это позволяет блокировать только те транзакции, которые собираются изменить данные. Благодаря мультиверсионности postgres соблюдает первые три принципа ACID.

### Хранение данных
Postgres хранит данные на дисках HDD или SSD, поэтому доступ к информации может быть не всегда быстрый. Postgres общается с дисками через файловую систему. Для того, чтобы избежать постоянного чтения информации с диска, файловая система сохраняет данные в кеш. Так что мы имеем локальный кеш у подпроцессов, общий кеш postgres и кеш файловой системы. Можно отметить, что даже изменение данных не происходит сразу на дисках, а какое-то время хранится в общем кеше postgres. Чтобы не потерять изменения, а также придерживаться 4 принципа ACID, postgres ведет журнал WAL, куда складывать все произведенные операции. При сбое postgres прочтет журнал и выполнит операции, которые еще не были выполнены.