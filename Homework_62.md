# Домашнее задание к занятию "6.2. SQL"

## Задача 1

Используя docker поднимите инстанс PostgreSQL (версию 12) c 2 volume, 
в который будут складываться данные БД и бэкапы.

Приведите получившуюся команду или docker-compose манифест.
### Ответ
```
version: "3.7"
services:
  postgres:
    image: postgres:12
   environment:
      POSTGRES_USER: "root"
      POSTGRES_PASSWORD: "12345678"
    volumes:
      - /home/diveslon/homework62/postgres/data:/data
      - /home/diveslon/homework62/postgres/backup:/backup
    ports:
      - "5432:5432"
```      

## Задача 2

В БД из задачи 1: 
- создайте пользователя test-admin-user и БД test_db
- в БД test_db создайте таблицу orders и clients (спeцификация таблиц ниже)
- предоставьте привилегии на все операции пользователю test-admin-user на таблицы БД test_db
- создайте пользователя test-simple-user  
- предоставьте пользователю test-simple-user права на SELECT/INSERT/UPDATE/DELETE данных таблиц БД test_db

Таблица orders:
- id (serial primary key)
- наименование (string)
- цена (integer)

Таблица clients:
- id (serial primary key)
- фамилия (string)
- страна проживания (string, index)
- заказ (foreign key orders)

Приведите:
- итоговый список БД после выполнения пунктов выше,
- описание таблиц (describe)
- SQL-запрос для выдачи списка пользователей с правами над таблицами test_db
- список пользователей с правами над таблицами test_db

### Ответ
```
1. SELECT datname FROM pg_database;

datname  |
---------+
postgres |
root     |
template1|
template0|
test_db  |
testdb1  |

2. 
SELECT column_name, data_type FROM INFORMATION_SCHEMA.columns WHERE table_name = 'clients';
column_name|data_type|
-----------+---------+
id         |integer  |
family     |text     |
country    |text     |
order_id   |integer  |

SELECT column_name, data_type FROM INFORMATION_SCHEMA.columns WHERE table_name = 'orders';
column_name|data_type|
-----------+---------+
id         |integer  |
name       |text     |
price      |integer  |

3.  SELECT * FROM information_schema.table_privileges WHERE table_name = 'orders' or table_name = 'clients';
4.
grantor|grantee         |table_catalog|table_schema|table_name|privilege_type|is_grantable|with_hierarchy|
-------+----------------+-------------+------------+----------+--------------+------------+--------------+
root   |root            |test_db      |public      |orders    |INSERT        |YES         |NO            |
root   |root            |test_db      |public      |orders    |SELECT        |YES         |YES           |
root   |root            |test_db      |public      |orders    |UPDATE        |YES         |NO            |
root   |root            |test_db      |public      |orders    |DELETE        |YES         |NO            |
root   |root            |test_db      |public      |orders    |TRUNCATE      |YES         |NO            |
root   |root            |test_db      |public      |orders    |REFERENCES    |YES         |NO            |
root   |root            |test_db      |public      |orders    |TRIGGER       |YES         |NO            |
root   |test_admin_user |test_db      |public      |orders    |INSERT        |NO          |NO            |
root   |test_admin_user |test_db      |public      |orders    |SELECT        |NO          |YES           |
root   |test_admin_user |test_db      |public      |orders    |UPDATE        |NO          |NO            |
root   |test_admin_user |test_db      |public      |orders    |DELETE        |NO          |NO            |
root   |test_admin_user |test_db      |public      |orders    |TRUNCATE      |NO          |NO            |
root   |test_admin_user |test_db      |public      |orders    |REFERENCES    |NO          |NO            |
root   |test_admin_user |test_db      |public      |orders    |TRIGGER       |NO          |NO            |
root   |test_simple_user|test_db      |public      |orders    |INSERT        |NO          |NO            |
root   |test_simple_user|test_db      |public      |orders    |SELECT        |NO          |YES           |
root   |test_simple_user|test_db      |public      |orders    |UPDATE        |NO          |NO            |
root   |test_simple_user|test_db      |public      |orders    |DELETE        |NO          |NO            |
root   |root            |test_db      |public      |clients   |INSERT        |YES         |NO            |
root   |root            |test_db      |public      |clients   |SELECT        |YES         |YES           |
root   |root            |test_db      |public      |clients   |UPDATE        |YES         |NO            |
root   |root            |test_db      |public      |clients   |DELETE        |YES         |NO            |
root   |root            |test_db      |public      |clients   |TRUNCATE      |YES         |NO            |
root   |root            |test_db      |public      |clients   |REFERENCES    |YES         |NO            |
root   |root            |test_db      |public      |clients   |TRIGGER       |YES         |NO            |
root   |test_admin_user |test_db      |public      |clients   |INSERT        |NO          |NO            |
root   |test_admin_user |test_db      |public      |clients   |SELECT        |NO          |YES           |
root   |test_admin_user |test_db      |public      |clients   |UPDATE        |NO          |NO            |
root   |test_admin_user |test_db      |public      |clients   |DELETE        |NO          |NO            |
root   |test_admin_user |test_db      |public      |clients   |TRUNCATE      |NO          |NO            |
root   |test_admin_user |test_db      |public      |clients   |REFERENCES    |NO          |NO            |
root   |test_admin_user |test_db      |public      |clients   |TRIGGER       |NO          |NO            |
root   |test_simple_user|test_db      |public      |clients   |INSERT        |NO          |NO            |
root   |test_simple_user|test_db      |public      |clients   |SELECT        |NO          |YES           |
root   |test_simple_user|test_db      |public      |clients   |UPDATE        |NO          |NO            |
root   |test_simple_user|test_db      |public      |clients   |DELETE        |NO          |NO            |
```

## Задача 3

Используя SQL синтаксис - наполните таблицы следующими тестовыми данными:

Таблица orders

|Наименование|цена|
|------------|----|
|Шоколад| 10 |
|Принтер| 3000 |
|Книга| 500 |
|Монитор| 7000|
|Гитара| 4000|

Таблица clients

|ФИО|Страна проживания|
|------------|----|
|Иванов Иван Иванович| USA |
|Петров Петр Петрович| Canada |
|Иоганн Себастьян Бах| Japan |
|Ронни Джеймс Дио| Russia|
|Ritchie Blackmore| Russia|

Используя SQL синтаксис:
- вычислите количество записей для каждой таблицы 
- приведите в ответе:
    - запросы 
    - результаты их выполнения.

### Ответ
```
select count(*) from clients;
count|
-----+
    5|

select count(*) from orders;
count|
-----+
    5|

```

## Задача 4

Часть пользователей из таблицы clients решили оформить заказы из таблицы orders.

Используя foreign keys свяжите записи из таблиц, согласно таблице:

|ФИО|Заказ|
|------------|----|
|Иванов Иван Иванович| Книга |
|Петров Петр Петрович| Монитор |
|Иоганн Себастьян Бах| Гитара |

Приведите SQL-запросы для выполнения данных операций.

Приведите SQL-запрос для выдачи всех пользователей, которые совершили заказ, а также вывод данного запроса.
 
Подсказк - используйте директиву `UPDATE`.

### Ответ
```
Поскольку поле индекса связи является обязательным, то добавил в таблицу заказов запись НЕТ Заказа - она получила индекс 6. 
insert into orders(name,price) values('НЕТ ЗАКАЗА',0);
Далее свяжем заказы, но сначала сделаем так, что ни у кого нет заказов
update clients set order_id = 6 where clients.id > 0;
update clients set order_id = 3 where clients.id = 1;
update clients set order_id = 4 where clients.id = 2;
update clients set order_id = 5 where clients.id = 3;

```

## Задача 5

Получите полную информацию по выполнению запроса выдачи всех пользователей из задачи 4 
(используя директиву EXPLAIN).

Приведите получившийся результат и объясните что значат полученные значения.

## Задача 6

Создайте бэкап БД test_db и поместите его в volume, предназначенный для бэкапов (см. Задачу 1).

Остановите контейнер с PostgreSQL (но не удаляйте volumes).

Поднимите новый пустой контейнер с PostgreSQL.

Восстановите БД test_db в новом контейнере.

Приведите список операций, который вы применяли для бэкапа данных и восстановления. 
