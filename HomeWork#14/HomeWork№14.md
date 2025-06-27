# Домашняя работа № 14 | Репликация: реализовать свой миникластер.

## Шаг 1. Подготовка и первоначальная настройка

Установил PostgreSQL 16 с дефолтными настрокайми

```
postgres=# select version();
PostgreSQL 16.9 (Ubuntu 16.9-0ubuntu0.24.04.1) on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 13.3.0-6ubuntu2~24.04) 13.3.0, 64-bit
```
Создаем инстансы (Дополнительные кластеры instance2, instance3, instance4)

```
sudo pg_createcluster 16 instance2
sudo pg_createcluster 16 instance3
sudo pg_createcluster 16 instance4

Creating new PostgreSQL cluster 16/instance2 ...
/usr/lib/postgresql/16/bin/initdb -D /var/lib/postgresql/16/instance2 --auth-local peer --auth-host scram-sha-256 --no-instructions
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "ru_RU.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "russian".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/16/instance2 ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Europe/Moscow
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok
Ver Cluster   Port Status Owner    Data directory                   Log file
16  instance2 5433 down   postgres /var/lib/postgresql/16/instance2 /var/log/postgresql/postgresql-16-instance2.log

Creating new PostgreSQL cluster 16/instance3 ...
/usr/lib/postgresql/16/bin/initdb -D /var/lib/postgresql/16/instance3 --auth-local peer --auth-host scram-sha-256 --no-instructions
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "ru_RU.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "russian".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/16/instance3 ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Europe/Moscow
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok
Ver Cluster   Port Status Owner    Data directory                   Log file
16  instance3 5434 down   postgres /var/lib/postgresql/16/instance3 /var/log/postgresql/postgresql-16-instance3.log

Creating new PostgreSQL cluster 16/instance4 ...
/usr/lib/postgresql/16/bin/initdb -D /var/lib/postgresql/16/instance4 --auth-local peer --auth-host scram-sha-256 --no-instructions
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "ru_RU.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "russian".

Data page checksums are disabled.

fixing permissions on existing directory /var/lib/postgresql/16/instance4 ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting default time zone ... Europe/Moscow
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok
Ver Cluster   Port Status Owner    Data directory                   Log file
16  instance4 5435 down   postgres /var/lib/postgresql/16/instance4 /var/log/postgresql/postgresql-16-instance4.log
```

Запускаем наши созданеые инстансы, проверяем работу:

```
student@studentPC:~$ sudo pg_ctlcluster 16 instance2 start
student@studentPC:~$ sudo pg_ctlcluster 16 instance3 start
student@studentPC:~$ sudo pg_ctlcluster 16 instance4 start

student@studentPC:~$ pg_lsclusters
Ver Cluster   Port Status Owner    Data directory                   Log file
16  instance2 5433 online postgres /var/lib/postgresql/16/instance2 /var/log/postgresql/postgresql-16-instance2.log
16  instance3 5434 online postgres /var/lib/postgresql/16/instance3 /var/log/postgresql/postgresql-16-instance3.log
16  instance4 5435 online postgres /var/lib/postgresql/16/instance4 /var/log/postgresql/postgresql-16-instance4.log
16  main      5432 online postgres /var/lib/postgresql/16/main      /var/log/postgresql/postgresql-16-main.log
```
Вот условные обозначение нашего кластера:

```
sudo -u postgres psql -p5432 - ВМ1
sudo -u postgres psql -p5433 - ВМ2
sudo -u postgres psql -p5434 - ВМ3
```

## Создадим на каждом инстансе 2 таблицы: test и test2:

```
Проделываем на каждом инстансе

create database test_replication;
\c test_replication;
test_replication=# create table test (id int, descr text);
CREATE TABLE
test_replication=# create table test2(id int, descr text);
CREATE TABLE
```

На 1 BM создадим таблицу на запись и публикацию для нее:

```
Настроим логическую репликацию на ВМ1 и создаем публикацию таблицы test:

sudo -u postgres psql -p5432 (1-ая ВМ)

ALTER SYSTEM SET wal_level = logical;

sudo pg_ctlcluster 16 main restart;

show wal_level;

postgres=# show wal_level;
 wal_level 
-----------
 logical

\c test_replication;

CREATE PUBLICATION test_publication FOR TABLE test;
```

На 2 ВМ подпишемся на публикацию таблицы test с 1 ВМ: (Использую атрибут copy_data=false | Чтобы таблицы не инициализировались и хранили только изменения)

```
sudo -u postgres psql -p5433 (2-ая ВМ)

ALTER SYSTEM SET wal_level = logical;

sudo pg_ctlcluster 16 instance2 restart;

show wal_level;

postgres=# show wal_level;
 wal_level 
-----------
 logical

\c test_replication;

Создадим подписку на таблицу ВМ1.test

CREATE SUBSCRIPTION test_subscription CONNECTION 'host=localhost port=5432 user=postgres password=qwerty dbname=test_replication' PUBLICATION test_publication WITH (copy_data = false);

ALTER TABLE test REPLICA IDENTITY FULL; - UPDATE
```

На 1 ВМ внесем значения в таблицу и проверим информацию на ВМ2

```
sudo -u postgres psql -p5432 (1-ая ВМ)

\c test_replication

insert into test(id, descr) values(1, 'example');

insert into test(id, descr) values(2, 'example');

insert into test(id, descr) values(3, 'example');

update test set descr = 'example-update' where id >1;

select * from test;

| id | descr | | --- | ----------- | | 1 | example | | 2 | example-update | | 3 | example-update |

Проверяем данные на 2-ой ВМ

select * from test;

| id | descr | | --- | ----------- | | 1 | example | | 2 | example-update | | 3 | example-update |

Данные приминились
```

На ВМ2 возьмем ранее созданную таблицу test2 для записи и создадим для нее публикацию для ВМ1

```
sudo -u postgres psql -p5433 (2-ая ВМ)

CREATE PUBLICATION test2_publication FOR TABLE test2;

На ВМ1 создадим подписку на таблицу test2 ВМ2

sudo -u postgres psql -p5432 (1-ая ВМ)

CREATE SUBSCRIPTION test_subscription CONNECTION 'host=localhost port=5433 user=postgres password=qwerty dbname=test_replication' PUBLICATION test2_publication WITH (copy_data = false);

ALTER TABLE test2 REPLICA IDENTITY FULL; - UPDATE

insert into test2(id, descr) values(1, 'example2');

insert into test2(id, descr) values(2, 'example2');

update test2 set descr = 'example2-update' where id =2 ;

select * from test2;

| id | descr | | --- | ----- | | 1 | example2 | | 2 | example2-update|

Провереям данные в таблице test2 для ВМ1

sudo -u postgres psql -p5432 (1-ая ВМ)

select * from test2;

| id | descr | | --- | ----- | | 1 | example2 | | 2 | example2-update|
```
## 3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).

```
CREATE SUBSCRIPTION test3_subscription_3 CONNECTION 'host=localhost port=5432 user=postgres password=qwerty dbname=test_replication' PUBLICATION test_publication WITH (copy_data = false);

CREATE SUBSCRIPTION test4_subscription_4 CONNECTION 'host=localhost port=5433 user=postgres password=qwerty dbname=test_replication' PUBLICATION test2_publication WITH (copy_data = false);
```
Тестируем, На ВМ 1 и 2 внесем изменения и проверим их на ВМ3: Выполним изменения на ВМ1

```
sudo -u postgres psql -p5432 (1-ая ВМ)

insert into test(id, descr) values(100, 'example3');

insert into test(id, descr) values(101, 'example3');

select * from test;

| id | descr | | --- | ----------- | | 1 | example | | 2 | example-update | | 3 | example-update | | 100 | example3 | | 101 | example3 |
```

Пробуем изменить данные на ВМ2

```
sudo -u postgres psql -p5433 (2-ая ВМ)

nsert into test2(id, descr) values(200, 'example2');

insert into test2(id, descr) values(201, 'example2');

select * from test2;

| id | descr | | --- | ------------ | | 1 | example2 | | 2 | example2-update | | 200 | example2 | | 201 | example2 |
```

Тестируем 3-ю ВМ

```
sudo -u postgres psql -p5434 (3-ая ВМ)

select * from test; 

| id | descr | | --- | ----------- | | 100 | example3 | | 101 | example3 |

select * from test2; 

| id | descr | | --- | ------------ | | 200 | example2 | | 201 |example2 |
```

Получилось, данные были среплицированны с 1-ой и 2-ой ВМ на 3-ю ВМ