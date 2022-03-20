# Домашнее задание к занятию "6.4. PostgreSQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 13). Данные БД сохраните в volume.

Подключитесь к БД PostgreSQL используя `psql`.

Воспользуйтесь командой `\?` для вывода подсказки по имеющимся в `psql` управляющим командам.

**Найдите и приведите** управляющие команды для:
- вывода списка БД
- подключения к БД
- вывода списка таблиц
- вывода описания содержимого таблиц
- выхода из psql

### Ответ
- вывода списка БД
```
\l[+]   [PATTERN]      list databases
```
- подключения к БД
```
\c[onnect] {[DBNAME|- USER|- HOST|- PORT|-] | conninfo}
                         connect to new database (currently "root")
```
- вывода списка таблиц
```
\dt[S+] [PATTERN]      list tables
```
- вывода описания содержимого таблиц
```
Informational
  (options: S = show system objects, + = additional detail)
\d[S+]  NAME           describe table, view, sequence, or index
```
- выхода из psql
```
\q                     quit psql
```

## Задача 2

Используя `psql` создайте БД `test_database`.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-04-postgresql/test_data).

Восстановите бэкап БД в `test_database`.

Перейдите в управляющую консоль `psql` внутри контейнера.

Подключитесь к восстановленной БД и проведите операцию ANALYZE для сбора статистики по таблице.

Используя таблицу [pg_stats](https://postgrespro.ru/docs/postgresql/12/view-pg-stats), найдите столбец таблицы `orders` 
с наибольшим средним значением размера элементов в байтах.

**Приведите в ответе** команду, которую вы использовали для вычисления и полученный результат.
### Ответ
```
root=# create database test_database;
root=# \l
                               List of databases
     Name      | Owner | Encoding |  Collate   |   Ctype    | Access privileges 
---------------+-------+----------+------------+------------+-------------------
 postgres      | root  | UTF8     | en_US.utf8 | en_US.utf8 | 
 root          | root  | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0     | root  | UTF8     | en_US.utf8 | en_US.utf8 | =c/root          +
               |       |          |            |            | root=CTc/root
 template1     | root  | UTF8     | en_US.utf8 | en_US.utf8 | =c/root          +
               |       |          |            |            | root=CTc/root
 test_database | root  | UTF8     | en_US.utf8 | en_US.utf8 | 
(5 rows)
```

```
root@20df11152f5f:/backup# psql test_database < /backup/test_dump.sql 
```

```
root=# \c test_database
You are now connected to database "test_database" as user "root".
test_database=# ANALYZE VERBOSE orders;
INFO:  analyzing "public.orders"
INFO:  "orders": scanned 1 of 1 pages, containing 8 live rows and 8 dead rows; 8 rows in sample, 8 estimated total rows
ANALYZE
test_database=#
```

```
root=# 

test_database=# select attname from pg_stats where tablename = 'orders' and avg_width = (select max(avg_width) from pg_stats where tablename = 'orders');
 attname 
---------
 title
(1 row)

test_database=#
```

## Задача 3

Архитектор и администратор БД выяснили, что ваша таблица orders разрослась до невиданных размеров и
поиск по ней занимает долгое время. Вам, как успешному выпускнику курсов DevOps в нетологии предложили
провести разбиение таблицы на 2 (шардировать на orders_1 - price>499 и orders_2 - price<=499).

Предложите SQL-транзакцию для проведения данной операции.

Можно ли было изначально исключить "ручное" разбиение при проектировании таблицы orders?

### Ответ

```
test_database=# create table orders_1 ( check ( price > 499 )) inherits(orders);
CREATE TABLE
test_databaseе=# create table orders_2 ( check ( price <= 499 )) inherits(orders);
CREATE TABLE
```
Можно было изначально исключить "ручное" разбиение при проектировании таблицы orders, спроектировав таблицу orders изначально разбитую на нужное количество таблиц, согласно приведённым запросам на создание шардированных таблиц. При вводе данных мы работаем с orders как с единой таблицой, а данные автоматически  раскладываются в нужную таблицу, согласно check.

## Задача 4

Используя утилиту `pg_dump` создайте бекап БД `test_database`.

Как бы вы доработали бэкап-файл, чтобы добавить уникальность значения столбца `title` для таблиц `test_database`?

### Ответ
```
root@20df11152f5f:/# pg_dump test_database > /backup/test_new.dump 
root@20df11152f5f:/# cd /backup/
root@20df11152f5f:/backup# ls
test_dump.sql  test_new.dump
```
Чтобы добавить уникальность значения столбца `title`, я бы заменил создание таблицы
```
CREATE TABLE public.orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL,
    price integer DEFAULT 0
);
```
на
```
CREATE TABLE public.orders (
    id integer NOT NULL,
    title character varying(80) NOT NULL UNIQUE,
    price integer DEFAULT 0
);
```
