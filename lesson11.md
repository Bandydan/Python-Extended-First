# Базы данных. Основы PostgreSQL

## Введение

`PSQL` относится к **реляционным СУБД (система управления базами данных)**, что значит, что в основе проектирования и построения данных лежат связи между ними, и именно связями между данными необходимо руководствоваться при проектировании баз данных. 

Базы данных предназначены для работы с информацией: для хранения и сохранения, поиска и редактирования данных, подчиненных какой-то структуре.

Крупнейшей единицей хранения данных является база данных - набор таблиц, связей между ними и прочих элементов - триггеров, представлений, временных таблиц и т.п.

в **СУБД** может быть сколько угодно баз данных.

Основой одной базы данных является таблица. Внешне она напоминает табличку в экселе. У таблицы в "заголовке" находятся имена полей, а каждый "ряд", называемый в теории баз данных **записью**, содержит одну запись значения для каждого поля. Если таблица Student, к примеру, содержит имена студентов(name) и их возраст(age), то говорят, что в таблице Student **поля** name и age. Каждый студент, записанный в таблицу, является одной записью, и выглядит это примерно так:

**Student**

|name |age|
|:----|:-:|
|Mark |21 |
|Jeny |22 |
|Tom  |21 |

Рассмотрим вопросы создания базы данных и таблиц поподробнее.

## Подключение к СУБД PostgreSQL

Если в системе установлена СУБД PostgreSQL, можно запустить соответствующее консольное приложение PostgreSQL, которое позволяет выполнять все доступные операции с базами данных.

PostgreSQL не умеет работать без базы данных. Еще до подключения к СУБД PostgreSQL необходимо создать ей базу данных или выбрать одну их существующих. Для просмотра существующих баз данных есть внешняя команда в консоли: 

```bash
➜ psql -l
                         List of databases
   Name    | Owner | Encoding | Collate | Ctype | Access privileges
-----------+-------+----------+---------+-------+-------------------
 postgres  | bandy | UTF8     | C       | C     |
 template0 | bandy | UTF8     | C       | C     | =c/bandy         +
 template1 | bandy | UTF8     | C       | C     | =c/bandy         +
 test      | bandy | UTF8     | C       | C     |
(4 rows)

```

Консольная команда для создания базы данных в PostgreSQL.

```bash
createdb test
```
После подключаемся к ней:

```bash
psql test
```
Общий вид команды подключения:

```
psql -U {{ user }} -d {{ dbname }} -h {{ host }}
```

На всякий случай укажу команду создания специального пользователя для работы с PostgreSQL со всеми возможностями:

```sql
# Создание пользователя с паролем
CREATE USER postgres WITH SUPERUSER PASSWORD 'password';
# Передача прав на основную настроечную базу данных новому пользователю
ALTER DATABASE postgres OWNER TO postgres;
# (ОПАСНО) Удаление пользователя, под которым зашли (ОПАСНО)
DROP USER USER_NAME;
```

## Создание баз данных и таблиц
  
Создание своей базы данных изнутри консоли psql - дело несложное:
  
```sql
create database test;
CREATE DATABASE
```
База создана. Переключимся в нее и добавим к ней первую табличку, тут будет немного посложнее:
  
```sql
\c test
You are now connected to database "test" as user "bandy".

create table cars(id serial, brand text not null);
CREATE TABLE
```

[Типы данных в psql](https://www.tutorialspoint.com/postgresql/postgresql_data_types.htm)

  
Посмотрим на результирующую таблицу:

```sql
\d cars
                            Table "public.cars"
 Column |  Type   | Collation | Nullable |             Default
--------+---------+-----------+----------+----------------------------------
 id     | integer |           | not null | nextval('cars_id_seq'::regclass)
 brand  | text    |           | not null |
```

Команда `\d` (сокращение от `describe`) с именем таблицы выводит краткое описание таблицы.


## Выбор базы данных, просмотр таблиц

Итак, мы в консоли `psql`. Просмотрим список существующих баз данных.
Команда `\l` показывает все существующие и доступные вашему пользователю базы данных.

Далее мы на примере встроенной базы данных разберемся, как выбирать конкретную базу данных и знакомиться с ее таблицами:

```sql
\c postgres
You are now connected to database "postgres" as user "bandy".
```

Командой `\c` пользователь говорит СУБД о том, что хочет работать с базой данных `postgres`, одноименной с самой СУБД. Далее, аналогично команде `l` для демонстрации доступных баз данных, применима команду `\dt` для демонстрации списка доступных таблиц.


## Другие операции с таблицами и базами данных

Для изменения существующей таблицы есть команда `alter table`:

```sql
alter table cars add column model varchar(100) not null;
ALTER TABLE

\d cars
                                    Table "public.cars"
 Column |          Type          | Collation | Nullable |             Default
--------+------------------------+-----------+----------+----------------------------------
 id     | integer                |           | not null | nextval('cars_id_seq'::regclass)
 brand  | text                   |           | not null |
 model  | character varying(100) |           | not null |
```

Мы добавили к таблице машин колонку с моделью машины, использовав команду `alter table`. Тем же путем удалим эту колонку:

```sql
alter table cars drop column model;
ALTER TABLE
\d cars
                            Table "public.cars"
 Column |  Type   | Collation | Nullable |             Default
--------+---------+-----------+----------+----------------------------------
 id     | integer |           | not null | nextval('cars_id_seq'::regclass)
 brand  | text    |           | not null |
```

[Команда alter table](http://www.postgresqltutorial.com/postgresql-alter-table/) довольно сложна, но и многое позволяет. Команда, как и при создании таблицы, позволяет добавлять и удалять колонки, менять их самыми различными способами, добавлять на них различные индексы, вешать ключи на таблицы и удалять их и так далее.

Для удаления таблиц есть две интересные команды:

`drop table` с именем таблицы просто удаляет таблицу.
`truncate table` с именем таблицы удаляет только содержимое таблицы, не меняя ее структуры, но пересоздавая первичный ключ, так что он снова будет добавлять "с единички".

База данных удаляется командой `drop database` с именем таблицы и точкой запятой в конце. 


### Полезные cсылки

[Домашнее задание](hw11.md)

[Следующий Урок](12_querries.md)