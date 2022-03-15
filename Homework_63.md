# Домашнее задание к занятию "6.3. MySQL"

## Введение

Перед выполнением задания вы можете ознакомиться с 
[дополнительными материалами](https://github.com/netology-code/virt-homeworks/tree/master/additional/README.md).

## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/master/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с данным контейнером.

### Ответ

```
1. Статус MySQL

mysql> \s
--------------
mysql  Ver 8.0.28 for Linux on x86_64 (MySQL Community Server - GPL)

Connection id:		9
Current database:	
Current user:		root@localhost
SSL:			Not in use
Current pager:		stdout
Using outfile:		''
Using delimiter:	;
Server version:		8.0.28 MySQL Community Server - GPL
Protocol version:	10
Connection:		Localhost via UNIX socket
Server characterset:	utf8mb4
Db     characterset:	utf8mb4
Client characterset:	latin1
Conn.  characterset:	latin1
UNIX socket:		/var/run/mysqld/mysqld.sock
Binary data as:		Hexadecimal
Uptime:			4 min 11 sec

Threads: 2  Questions: 5  Slow queries: 0  Opens: 117  Flush tables: 3  Open tables: 36  Queries per second avg: 0.019
--------------

mysql> 

2. Количество записей с `price` > 300.

mysql> select count(*) from orders where price > 300;
+----------+
| count(*) |
+----------+
|        1 |
+----------+
1 row in set (0.00 sec)

mysql> 
```

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней 
- количество попыток авторизации - 3 
- максимальное количество запросов в час - 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

### Ответ
```
Создание и настройка пользователя

mysql> CREATE USER 'test' IDENTIFIED WITH mysql_native_password BY 'test-pass' PASSWORD EXPIRE INTERVAL 180 DAY;
Quemysql> ALTER USER 'test' WITH MAX_QUERIES_PER_HOUR 100;
Query OK, 0 rows affected (0.02 sec)
ry OK, 0 rows affected (0.01 sec)
mysql> ALTER USER 'test' FAILED_LOGIN_ATTEMPTS 3;
Query OK, 0 rows affected (0.00 sec)
mysql> ALTER USER 'test' ATTRIBUTE '{"fname": "Pretty", "lname": "James"}';
Query OK, 0 rows affected (0.01 sec)
mysql> GRANT SELECT ON db.* TO 'test';
Query OK, 0 rows affected (0.04 sec)


Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` 
mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE user='test';
+------+------+---------------------------------------+
| USER | HOST | ATTRIBUTE                             |
+------+------+---------------------------------------+
| test | %    | {"fname": "Pretty", "lname": "James"} |
+------+------+---------------------------------------+
1 row in set (0.00 sec)

```

## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
- на `InnoDB`

### Ответ
```
mysql> ALTER TABLE orders ENGINE = MyISAM;
Query OK, 5 rows affected (0.04 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES WHERE TABLE_NAME = 'orders';
+------------+--------+
| TABLE_NAME | ENGINE |
+------------+--------+
| orders     | MyISAM |
+------------+--------+
1 row in set (0.01 sec)

mysql> SHOW PROFILES;
+----------+------------+-------------------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                                 |
+----------+------------+-------------------------------------------------------------------------------------------------------+
|        1 | 0.00199675 | SELECT ENGINE FROM information_schema.TABLES WHERE TABLE_SCHEMA = 'test_db' AND TABLE_NAME = 'orders' |
|        2 | 0.00245400 | SELECT * FROM information_schema.TABLES WHERE TABLE_NAME = 'orders'                                   |
|        3 | 0.00189700 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES WHERE TABLE_NAME = 'orders'                  |
|        4 | 0.03264475 | ALTER TABLE orders ENGINE = MyISAM                                                                    |
|        5 | 0.00185650 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES WHERE TABLE_NAME = 'orders'                  |
+----------+------------+-------------------------------------------------------------------------------------------------------+
5 rows in set, 1 warning (0.00 sec)

mysql> ALTER TABLE orders ENGINE = InnoDB;
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SHOW PROFILES;
+----------+------------+-------------------------------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                                                 |
+----------+------------+-------------------------------------------------------------------------------------------------------+
|        1 | 0.00199675 | SELECT ENGINE FROM information_schema.TABLES WHERE TABLE_SCHEMA = 'test_db' AND TABLE_NAME = 'orders' |
|        2 | 0.00245400 | SELECT * FROM information_schema.TABLES WHERE TABLE_NAME = 'orders'                                   |
|        3 | 0.00189700 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES WHERE TABLE_NAME = 'orders'                  |
|        4 | 0.03264475 | ALTER TABLE orders ENGINE = MyISAM                                                                    |
|        5 | 0.00185650 | SELECT TABLE_NAME, ENGINE FROM information_schema.TABLES WHERE TABLE_NAME = 'orders'                  |
|        6 | 0.02276850 | ALTER TABLE orders ENGINE = InnoDB                                                                    |
+----------+------------+-------------------------------------------------------------------------------------------------------+
6 rows in set, 1 warning (0.00 sec)


```

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных
- Нужна компрессия таблиц для экономии места на диске
- Размер буффера с незакомиченными транзакциями 1 Мб
- Буффер кеширования 30% от ОЗУ
- Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf`.

### Ответ
```
root@06bec90bdfd8:/etc/mysql# cat my.cnf
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
# 
# This program is free software; you can redistribute it and/or modify
# This program is free software; you can redistribute it and/or modify

# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA

#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html


[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= NULL


innodb_log_file_size = 100Mb
innodb_file_per_table = ON
innodb_flush_method = O_DSYNC
innodb_log_buffer_size = 1Mb
innodb_buffer_pool_size = 2.4Gb

```

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
