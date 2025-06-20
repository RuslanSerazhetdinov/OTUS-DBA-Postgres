# Домашняя работа № 13 | Применить логический бэкап. Восстановиться из бэкапа.


## Шаг 1. На созданой ВМ на базе ОС Linux Ubuntu был установлен 16-ый postgresql, создаю БД и схему для выполнения данного задания.

```
postgres=# create database task_backup;
CREATE DATABASE
postgres=# \c task_backup;
You are now connected to database "task_backup" as user "postgres".
task_backup=# create schema test_backup;
CREATE SCHEMA
```

В схеме создадим табличку *test_backup.document_template* и дополним ее скриптом:

```
task_backup=# CREATE TABLE test_backup.document_template(
    ID INTEGER NOT NULL,
    NAME TEXT,
    SHORT_DESCRIPTION TEXT,
    AUTHOR TEXT,
    DESCRIPTION TEXT,
    CONTENT TEXT,
    LAST_UPDATED DATE,
    CREATED DATE);
CREATE TABLE
```

Генерируем рандомные 100 записей:

```
task_backup=# INSERT INTO test_backup.document_template(id,name, short_description, author, description,content, last_updated,created)
    SELECT id, 'name', md5(random()::text), 'name2'
    ,md5(random()::text),md5(random()::text)
    ,NOW() - '1 day'::INTERVAL * (RANDOM()::int * 100)
    ,NOW() - '1 day'::INTERVAL * (RANDOM()::int * 100 + 100)
    FROM generate_series(1,100) id;
INSERT 0 100
```

Селектим вывод:

```
task_backup=# select * from test_backup.document_template;

id  | name |        short_description         | author |           description            |             content              | last_updated |  created   
-----+------+----------------------------------+--------+----------------------------------+----------------------------------+--------------+------------
   1 | name | bde1503a83b3b472198773c2cda29cf4 | name2  | 3c5dfbe18f705daaacff4d1cfe0d4aa7 | d649b32a7099ec56686322b231ab176b | 2025-03-12   | 2025-03-12
   2 | name | c13f3a0a91bbf0d55446e620a3e28c34 | name2  | 3583b280eb25ad0dddeef78f395bd2a6 | 62371110ffaf4452942522eb2dd0fe74 | 2025-06-20   | 2024-12-02
   3 | name | c8c560036803126021a68a492dea6157 | name2  | 13884371f86053c52772b923f6c6b00c | 15eaaa00ee2c76593326f3884d702ca3 | 2025-06-20   | 2025-03-12
   4 | name | efe8ec333c1fea04d8b91925689974d4 | name2  | b428a2be590f6fe9e019a191d679d3da | 9e254cf9699db829ba05142d9fb35ed2 | 2025-06-20   | 2025-03-12
   5 | name | bfc2e518220c8540837a1aae97433dae | name2  | c8c758ff0d531af983d6429027f981ba | f3569034c17bc1ed26388bd7b6a7cab3 | 2025-06-20   | 2025-03-12
   6 | name | 1fb9a3c2a4449a3410c60e2f489c951e | name2  | 3415083fbd793270b4129d0e6428790c | b027feed6fd233f8e37f0758e24c80a7 | 2025-06-20   | 2025-03-12
   7 | name | 7e590b913b08c7eeb075a88ac4080595 | name2  | 731017d6d50d0eab7ddbaa52ec225fcb | 46269278662923b4ff5f7c72be1d434a | 2025-03-12   | 2025-03-12
   8 | name | 662b8ec7f55f39702f47c700283fd6c6 | name2  | 0c31c3ae09784b239df9882647262640 | e7111e6f40aabd27ad2ebb88c5d0a3d9 | 2025-06-20   | 2024-12-02
   9 | name | 45a727ca9471f16bbd83e2d1f4cac313 | name2  | 29365ef7a59a514f5a5c593481dc28e8 | 316d92014559e76ed548f7a1c0aeb129 | 2025-06-20   | 2024-12-02
  10 | name | 4892292f6cb8f04a39caa565f2be1d68 | name2  | 1246a1f416577970c48e44afc75f7c37 | f775f39183425166d787631b117bfc82 | 2025-06-20   | 2025-03-12
  11 | name | 16bf77bb79d136d7f8e086e39c3cac4e | name2  | 4b2679827ba9c77e625364769025151d | b7966b53abfe8926e4d7ee1ddee5bc1f | 2025-06-20   | 2024-12-02
  12 | name | fefa3ccc851536b2f4f943c93f2afc54 | name2  | 8a4299378faec496246b14a41f23cc8a | cfae5c4e6f3359e8149c7158110d8847 | 2025-06-20   | 2024-12-02
  13 | name | bdb22251ed7c64c3a2de83813eb49b83 | name2  | fc66b93c23b7d48e59805ec494ed1c49 | 083ad4f8fa0bbd7d5f12b044dea65bab | 2025-06-20   | 2025-03-12
  14 | name | 57e0228852eb517aa083ca2df480b84f | name2  | 420e1a17721fe6e2df5f25b7be02be23 | 4299ec217166eeb9eeed26704c19a9fc | 2025-03-12   | 2024-12-02
  16 | name | 3f0047e5b6c9d80e199b25359bf0b7dd | name2  | 5351468903c82658b0eb93125126d792 | 06bce04cb4e17f6144790f39814cb397 | 2025-03-12   | 2025-03-12
  17 | name | 7f0db9e91f85db1d60c4312de2b25150 | name2  | 22926577f4b3ff41c74b862b7875d230 | 8d6218d23f4809fd17495d56d66f7cc0 | 2025-03-12   | 2025-03-12
  18 | name | 0fcd655f80a02f319599aee5ef179a84 | name2  | dcb2d7bb08877113074c165b01676a51 | fc922886b36b7ece938eb994073ac841 | 2025-06-20   | 2024-12-02
  19 | name | 581815821398343713c6057277e18ecc | name2  | 1028cbebcfd4ed5ed8c94fd05187de7a | d7e73037b4c37d078dbe0806747931bf | 2025-06-20   | 2025-03-12
  20 | name | 4556c8def54ffd5149e241407b6765e1 | name2  | 042914ff7dd01e6f39bfcb8ba77d3d03 | fb0f5b5d76ef5f25c6655e5b763c23f2 | 2025-06-20   | 2024-12-02
  21 | name | 75c66fa6de552de5e4785eaf0a0c48df | name2  | b39e1975ed072ee5983d0bc460039893 | 2c5e14db6279ef6a1bb44751242ae01b | 2025-03-12   | 2025-03-12
  22 | name | 08c57eab0924294a33db9287323153e6 | name2  | 3f88621e832da9d44eabb80848ea39c9 | c0f2d3636e3ce2fab30c4de6fcd063b8 | 2025-06-20   | 2025-03-12
  23 | name | 01676adc3427ac558807c1dbf883438d | name2  | 8e1068cb68b3942204708aae85e070a7 | 5c2b8a37d6cca013bd5fb207f3f7ff52 | 2025-06-20   | 2025-03-12
  24 | name | 988f50316ff9771450ca9bb59c80c816 | name2  | 97beebe9a8d35e4bdfcf77f1513cf495 | 0988825f542d4525001a2b9a7086e3f8 | 2025-03-12   | 2025-03-12
  25 | name | a8b2c1b7073886f1b22aca510ec32dc9 | name2  | 7d5ed70353bdbb2792fc945e2d9a0e99 | 6ec6faa564f83cac4ac92398ececcae0 | 2025-06-20   | 2024-12-02
  26 | name | 8bde84852443973ecc1444e12d7f9e90 | name2  | f964dda89f82896913464fd3abb95346 | 44bc7aa3724a563b4f799a3c0cb1a5f1 | 2025-06-20   | 2025-03-12
  27 | name | e57a54b12b53a088aebc2b5bbe0d7e3f | name2  | 80b85069863d9aaa9860ea9d54060b8e | d36b0631809649ae2afcb84ce5b3e9ad | 2025-06-20   | 2025-03-12
  28 | name | 4c2aa890f6ff2e4549075ed6de25c44d | name2  | d4c31c65013942e3e99f8426d0e351f0 | cea469ed2055816b1c3e56fdc605e18f | 2025-03-12   | 2025-03-12
  29 | name | 1332e38bf5dc750b99cc6513542960a8 | name2  | 852e4792d21b650bd528780206477063 | 338deccab809e44f5d8f660adaeb1a2c | 2025-06-20   | 2025-03-12
  30 | name | 164c5078f5d5a90dbc78cc718754aef7 | name2  | fee594f5969a9841c0eabac75115a748 | 4c2bd32ce5f42593fffffc708fde79b3 | 2025-03-12   | 2025-03-12
  31 | name | 240a2fb3b896a9cb515b42bbe19e3aa0 | name2  | 8ebf00fcaf1bb487cc135c979a323757 | 4c9c1263c79986e20998b2e2e8090db8 | 2025-06-20   | 2025-03-12
  32 | name | ea28792c85dbde348bc1608fd3e05e7f | name2  | d6a0eb934444e0a9a9225bb8022b536f | 7e2394c5d239568d5831919348b799f9 | 2025-03-12   | 2025-03-12
  33 | name | f5b00d6e32a1c526c7af485e385543e3 | name2  | dbb1dd6ddb6b3c96ae72a44a8dd07edf | 631f74125db6e9217889851ce0a9c856 | 2025-06-20   | 2024-12-02
  34 | name | 51842df4de3b7da66c100a60a9547e7c | name2  | bc366a1cb380748d434f25bfafffa083 | dee3c9c4010b21fd95ab1c5daacb5a48 | 2025-03-12   | 2024-12-02
  35 | name | 0324c615f7a77fb86b4e076fcb39665a | name2  | 3c7c962560f46361f821c52d29578236 | 621014c24a5f6f0c646fba574709e166 | 2025-06-20   | 2025-03-12
  36 | name | 2d8e54322a472125ee2ad4f46c150e10 | name2  | 64757b9d6e384baaeaba0b9e7928126e | 54c64af831aee94d589849a0800e6214 | 2025-03-12   | 2025-03-12
  37 | name | 8e7bb26f72d23e2ec2094d2044198266 | name2  | e4209a959ca057e4681cbd3eb80cb306 | d9c9493f231826b1ccc5b2b2e6f63404 | 2025-03-12   | 2024-12-02

Дальше вывод вставлять не буду, инфо для наглядности :)
```

## Шаг 2. Создаем директорию для бекапов и выполняем логический бекап.

В рамках данного задания создам директорию на той же ВМ, но в реальных задачах лучше это делать на отдельном диске или другой ВМ (хранить бекапы на дисковых системах отдельно от БД)

```
postgres@studentPC:~$ cd /etc/postgresql
postgres@studentPC:/etc/postgresql$ mkdir backup
postgres@studentPC:/etc/postgresql$ ls 
16  backup
```

Содаем логический бекап командой *COPY*:

```
postgres=# \c task_backup;
You are now connected to database "task_backup" as user "postgres".
task_backup=# \copy test_backup.document_template to '/etc/postgresql/backup/backup_copy_test_backup.document_template.sql';
COPY 100
```

Смотрим что  получилось:

```
postgres@studentPC:/etc/postgresql$ less /etc/postgresql/backup/backup_copy_test_backup.document_template.sql

1       name    bde1503a83b3b472198773c2cda29cf4        name2   3c5dfbe18f705daaacff4d1cfe0d4aa7        d649b32a7099ec56686322b231ab176b        2025-03-12      2025-03-12
2       name    c13f3a0a91bbf0d55446e620a3e28c34        name2   3583b280eb25ad0dddeef78f395bd2a6        62371110ffaf4452942522eb2dd0fe74        2025-06-20      2024-12-02
3       name    c8c560036803126021a68a492dea6157        name2   13884371f86053c52772b923f6c6b00c        15eaaa00ee2c76593326f3884d702ca3        2025-06-20      2025-03-12
4       name    efe8ec333c1fea04d8b91925689974d4        name2   b428a2be590f6fe9e019a191d679d3da        9e254cf9699db829ba05142d9fb35ed2        2025-06-20      2025-03-12
5       name    bfc2e518220c8540837a1aae97433dae        name2   c8c758ff0d531af983d6429027f981ba        f3569034c17bc1ed26388bd7b6a7cab3        2025-06-20      2025-03-12
6       name    1fb9a3c2a4449a3410c60e2f489c951e        name2   3415083fbd793270b4129d0e6428790c        b027feed6fd233f8e37f0758e24c80a7        2025-06-20      2025-03-12
7       name    7e590b913b08c7eeb075a88ac4080595        name2   731017d6d50d0eab7ddbaa52ec225fcb        46269278662923b4ff5f7c72be1d434a        2025-03-12      2025-03-12
8       name    662b8ec7f55f39702f47c700283fd6c6        name2   0c31c3ae09784b239df9882647262640        e7111e6f40aabd27ad2ebb88c5d0a3d9        2025-06-20      2024-12-02
9       name    45a727ca9471f16bbd83e2d1f4cac313        name2   29365ef7a59a514f5a5c593481dc28e8        316d92014559e76ed548f7a1c0aeb129        2025-06-20      2024-12-02
10      name    4892292f6cb8f04a39caa565f2be1d68        name2   1246a1f416577970c48e44afc75f7c37        f775f39183425166d787631b117bfc82        2025-06-20      2025-03-12
11      name    16bf77bb79d136d7f8e086e39c3cac4e        name2   4b2679827ba9c77e625364769025151d        b7966b53abfe8926e4d7ee1ddee5bc1f        2025-06-20      2024-12-02
12      name    fefa3ccc851536b2f4f943c93f2afc54        name2   8a4299378faec496246b14a41f23cc8a        cfae5c4e6f3359e8149c7158110d8847        2025-06-20      2024-12-02
13      name    bdb22251ed7c64c3a2de83813eb49b83        name2   fc66b93c23b7d48e59805ec494ed1c49        083ad4f8fa0bbd7d5f12b044dea65bab        2025-06-20      2025-03-12
14      name    57e0228852eb517aa083ca2df480b84f        name2   420e1a17721fe6e2df5f25b7be02be23        4299ec217166eeb9eeed26704c19a9fc        2025-03-12      2024-12-02
15      name    9361f5ef16afc9cd85ba090d291d002e        name2   8c161e9d47589e3261f8c600a77c1cec        cef37aba3dedca51649277ce3ceabfe7        2025-03-12      2025-03-12
16      name    3f0047e5b6c9d80e199b25359bf0b7dd        name2   5351468903c82658b0eb93125126d792        06bce04cb4e17f6144790f39814cb397        2025-03-12      2025-03-12
17      name    7f0db9e91f85db1d60c4312de2b25150        name2   22926577f4b3ff41c74b862b7875d230        8d6218d23f4809fd17495d56d66f7cc0        2025-03-12      2025-03-12
18      name    0fcd655f80a02f319599aee5ef179a84        name2   dcb2d7bb08877113074c165b01676a51        fc922886b36b7ece938eb994073ac841        2025-06-20      2024-12-02
19      name    581815821398343713c6057277e18ecc        name2   1028cbebcfd4ed5ed8c94fd05187de7a        d7e73037b4c37d078dbe0806747931bf        2025-06-20      2025-03-12
20      name    4556c8def54ffd5149e241407b6765e1        name2   042914ff7dd01e6f39bfcb8ba77d3d03        fb0f5b5d76ef5f25c6655e5b763c23f2        2025-06-20      2024-12-02
21      name    75c66fa6de552de5e4785eaf0a0c48df        name2   b39e1975ed072ee5983d0bc460039893        2c5e14db6279ef6a1bb44751242ae01b        2025-03-12      2025-03-12
22      name    08c57eab0924294a33db9287323153e6        name2   3f88621e832da9d44eabb80848ea39c9        c0f2d3636e3ce2fab30c4de6fcd063b8        2025-06-20      2025-03-12
23      name    01676adc3427ac558807c1dbf883438d        name2   8e1068cb68b3942204708aae85e070a7        5c2b8a37d6cca013bd5fb207f3f7ff52        2025-06-20      2025-03-12
24      name    988f50316ff9771450ca9bb59c80c816        name2   97beebe9a8d35e4bdfcf77f1513cf495        0988825f542d4525001a2b9a7086e3f8        2025-03-12      2025-03-12
25      name    a8b2c1b7073886f1b22aca510ec32dc9        name2   7d5ed70353bdbb2792fc945e2d9a0e99        6ec6faa564f83cac4ac92398ececcae0        2025-06-20      2024-12-02
26      name    8bde84852443973ecc1444e12d7f9e90        name2   f964dda89f82896913464fd3abb95346        44bc7aa3724a563b4f799a3c0cb1a5f1        2025-06-20      2025-03-12
27      name    e57a54b12b53a088aebc2b5bbe0d7e3f        name2   80b85069863d9aaa9860ea9d54060b8e        d36b0631809649ae2afcb84ce5b3e9ad        2025-06-20      2025-03-12
28      name    4c2aa890f6ff2e4549075ed6de25c44d        name2   d4c31c65013942e3e99f8426d0e351f0        cea469ed2055816b1c3e56fdc605e18f        2025-03-12      2025-03-12
29      name    1332e38bf5dc750b99cc6513542960a8        name2   852e4792d21b650bd528780206477063        338deccab809e44f5d8f660adaeb1a2c        2025-06-20      2025-03-12
30      name    164c5078f5d5a90dbc78cc718754aef7        name2   fee594f5969a9841c0eabac75115a748        4c2bd32ce5f42593fffffc708fde79b3        2025-03-12      2025-03-12
31      name    240a2fb3b896a9cb515b42bbe19e3aa0        name2   8ebf00fcaf1bb487cc135c979a323757        4c9c1263c79986e20998b2e2e8090db8        2025-06-20      2025-03-12
32      name    ea28792c85dbde348bc1608fd3e05e7f        name2   d6a0eb934444e0a9a9225bb8022b536f        7e2394c5d239568d5831919348b799f9        2025-03-12      2025-03-12
33      name    f5b00d6e32a1c526c7af485e385543e3        name2   dbb1dd6ddb6b3c96ae72a44a8dd07edf        631f74125db6e9217889851ce0a9c856        2025-06-20      2024-12-02
34      name    51842df4de3b7da66c100a60a9547e7c        name2   bc366a1cb380748d434f25bfafffa083        dee3c9c4010b21fd95ab1c5daacb5a48        2025-03-12      2024-12-02
35      name    0324c615f7a77fb86b4e076fcb39665a        name2   3c7c962560f46361f821c52d29578236        621014c24a5f6f0c646fba574709e166        2025-06-20      2025-03-12
36      name    2d8e54322a472125ee2ad4f46c150e10        name2   64757b9d6e384baaeaba0b9e7928126e        54c64af831aee94d589849a0800e6214        2025-03-12      2025-03-12
37      name    8e7bb26f72d23e2ec2094d2044198266        name2   e4209a959ca057e4681cbd3eb80cb306        d9c9493f231826b1ccc5b2b2e6f63404        2025-03-12      2024-12-02
38      name    8cff8ef57b1cd63b48f0f429a5cbb104        name2   84b19ad603fe8fe2cb79fae680cb0136        a9ed1f647167866553ad02493b6e8154        2025-03-12      2024-12-02
39      name    742c192943f25691f6e37bfbf75d3b5b        name2   64e8ca30fe2b31810e994821526e7b95        1e521b0eee950341ca38bf951ddc44c6        2025-03-12      2025-03-12

Дальше вывод вставлять не буду, инфо для наглядности :)
```

## Шаг 3. Создаем новую таблицу и пробуем восстановится.

Создаем новую таблицу для восстановления:

```
task_backup=# CREATE TABLE test_backup.document_template_for_copy(
    ID INTEGER NOT NULL,
    NAME TEXT,
    SHORT_DESCRIPTION TEXT,
    AUTHOR TEXT,
    DESCRIPTION TEXT,
    CONTENT TEXT,
    LAST_UPDATED DATE,
    CREATED DATE
    );
CREATE TABLE
```

Фиксируем состояние до восстановления:

```
task_backup=# select * from test_backup.document_template_for_copy;
 id | name | short_description | author | description | content | last_updated | created 
----+------+-------------------+--------+-------------+---------+--------------+---------
(0 rows)
```

Пробуем восстановится из ранее созданого бекапа и селектим после восстановления:

```
task_backup=# \copy test_backup.document_template_for_copy from '/etc/postgresql/backup/backup_copy_test_backup.document_template.sql';
COPY 100

task_backup=# select * from test_backup.document_template_for_copy;

id  | name |        short_description         | author |           description            |             content              | last_updated |  created   
-----+------+----------------------------------+--------+----------------------------------+----------------------------------+--------------+------------
   1 | name | bde1503a83b3b472198773c2cda29cf4 | name2  | 3c5dfbe18f705daaacff4d1cfe0d4aa7 | d649b32a7099ec56686322b231ab176b | 2025-03-12   | 2025-03-12
   2 | name | c13f3a0a91bbf0d55446e620a3e28c34 | name2  | 3583b280eb25ad0dddeef78f395bd2a6 | 62371110ffaf4452942522eb2dd0fe74 | 2025-06-20   | 2024-12-02
   3 | name | c8c560036803126021a68a492dea6157 | name2  | 13884371f86053c52772b923f6c6b00c | 15eaaa00ee2c76593326f3884d702ca3 | 2025-06-20   | 2025-03-12
   4 | name | efe8ec333c1fea04d8b91925689974d4 | name2  | b428a2be590f6fe9e019a191d679d3da | 9e254cf9699db829ba05142d9fb35ed2 | 2025-06-20   | 2025-03-12
   5 | name | bfc2e518220c8540837a1aae97433dae | name2  | c8c758ff0d531af983d6429027f981ba | f3569034c17bc1ed26388bd7b6a7cab3 | 2025-06-20   | 2025-03-12
   6 | name | 1fb9a3c2a4449a3410c60e2f489c951e | name2  | 3415083fbd793270b4129d0e6428790c | b027feed6fd233f8e37f0758e24c80a7 | 2025-06-20   | 2025-03-12
   7 | name | 7e590b913b08c7eeb075a88ac4080595 | name2  | 731017d6d50d0eab7ddbaa52ec225fcb | 46269278662923b4ff5f7c72be1d434a | 2025-03-12   | 2025-03-12
   8 | name | 662b8ec7f55f39702f47c700283fd6c6 | name2  | 0c31c3ae09784b239df9882647262640 | e7111e6f40aabd27ad2ebb88c5d0a3d9 | 2025-06-20   | 2024-12-02
   9 | name | 45a727ca9471f16bbd83e2d1f4cac313 | name2  | 29365ef7a59a514f5a5c593481dc28e8 | 316d92014559e76ed548f7a1c0aeb129 | 2025-06-20   | 2024-12-02
  10 | name | 4892292f6cb8f04a39caa565f2be1d68 | name2  | 1246a1f416577970c48e44afc75f7c37 | f775f39183425166d787631b117bfc82 | 2025-06-20   | 2025-03-12
  11 | name | 16bf77bb79d136d7f8e086e39c3cac4e | name2  | 4b2679827ba9c77e625364769025151d | b7966b53abfe8926e4d7ee1ddee5bc1f | 2025-06-20   | 2024-12-02
  12 | name | fefa3ccc851536b2f4f943c93f2afc54 | name2  | 8a4299378faec496246b14a41f23cc8a | cfae5c4e6f3359e8149c7158110d8847 | 2025-06-20   | 2024-12-02
  13 | name | bdb22251ed7c64c3a2de83813eb49b83 | name2  | fc66b93c23b7d48e59805ec494ed1c49 | 083ad4f8fa0bbd7d5f12b044dea65bab | 2025-06-20   | 2025-03-12
  14 | name | 57e0228852eb517aa083ca2df480b84f | name2  | 420e1a17721fe6e2df5f25b7be02be23 | 4299ec217166eeb9eeed26704c19a9fc | 2025-03-12   | 2024-12-02
  15 | name | 9361f5ef16afc9cd85ba090d291d002e | name2  | 8c161e9d47589e3261f8c600a77c1cec | cef37aba3dedca51649277ce3ceabfe7 | 2025-03-12   | 2025-03-12
  16 | name | 3f0047e5b6c9d80e199b25359bf0b7dd | name2  | 5351468903c82658b0eb93125126d792 | 06bce04cb4e17f6144790f39814cb397 | 2025-03-12   | 2025-03-12
  17 | name | 7f0db9e91f85db1d60c4312de2b25150 | name2  | 22926577f4b3ff41c74b862b7875d230 | 8d6218d23f4809fd17495d56d66f7cc0 | 2025-03-12   | 2025-03-12
  18 | name | 0fcd655f80a02f319599aee5ef179a84 | name2  | dcb2d7bb08877113074c165b01676a51 | fc922886b36b7ece938eb994073ac841 | 2025-06-20   | 2024-12-02
  19 | name | 581815821398343713c6057277e18ecc | name2  | 1028cbebcfd4ed5ed8c94fd05187de7a | d7e73037b4c37d078dbe0806747931bf | 2025-06-20   | 2025-03-12
  20 | name | 4556c8def54ffd5149e241407b6765e1 | name2  | 042914ff7dd01e6f39bfcb8ba77d3d03 | fb0f5b5d76ef5f25c6655e5b763c23f2 | 2025-06-20   | 2024-12-02
  21 | name | 75c66fa6de552de5e4785eaf0a0c48df | name2  | b39e1975ed072ee5983d0bc460039893 | 2c5e14db6279ef6a1bb44751242ae01b | 2025-03-12   | 2025-03-12
  22 | name | 08c57eab0924294a33db9287323153e6 | name2  | 3f88621e832da9d44eabb80848ea39c9 | c0f2d3636e3ce2fab30c4de6fcd063b8 | 2025-06-20   | 2025-03-12
  23 | name | 01676adc3427ac558807c1dbf883438d | name2  | 8e1068cb68b3942204708aae85e070a7 | 5c2b8a37d6cca013bd5fb207f3f7ff52 | 2025-06-20   | 2025-03-12
  24 | name | 988f50316ff9771450ca9bb59c80c816 | name2  | 97beebe9a8d35e4bdfcf77f1513cf495 | 0988825f542d4525001a2b9a7086e3f8 | 2025-03-12   | 2025-03-12
  25 | name | a8b2c1b7073886f1b22aca510ec32dc9 | name2  | 7d5ed70353bdbb2792fc945e2d9a0e99 | 6ec6faa564f83cac4ac92398ececcae0 | 2025-06-20   | 2024-12-02
  26 | name | 8bde84852443973ecc1444e12d7f9e90 | name2  | f964dda89f82896913464fd3abb95346 | 44bc7aa3724a563b4f799a3c0cb1a5f1 | 2025-06-20   | 2025-03-12
  27 | name | e57a54b12b53a088aebc2b5bbe0d7e3f | name2  | 80b85069863d9aaa9860ea9d54060b8e | d36b0631809649ae2afcb84ce5b3e9ad | 2025-06-20   | 2025-03-12
  28 | name | 4c2aa890f6ff2e4549075ed6de25c44d | name2  | d4c31c65013942e3e99f8426d0e351f0 | cea469ed2055816b1c3e56fdc605e18f | 2025-03-12   | 2025-03-12
  29 | name | 1332e38bf5dc750b99cc6513542960a8 | name2  | 852e4792d21b650bd528780206477063 | 338deccab809e44f5d8f660adaeb1a2c | 2025-06-20   | 2025-03-12
  30 | name | 164c5078f5d5a90dbc78cc718754aef7 | name2  | fee594f5969a9841c0eabac75115a748 | 4c2bd32ce5f42593fffffc708fde79b3 | 2025-03-12   | 2025-03-12
  31 | name | 240a2fb3b896a9cb515b42bbe19e3aa0 | name2  | 8ebf00fcaf1bb487cc135c979a323757 | 4c9c1263c79986e20998b2e2e8090db8 | 2025-06-20   | 2025-03-12
  32 | name | ea28792c85dbde348bc1608fd3e05e7f | name2  | d6a0eb934444e0a9a9225bb8022b536f | 7e2394c5d239568d5831919348b799f9 | 2025-03-12   | 2025-03-12
  33 | name | f5b00d6e32a1c526c7af485e385543e3 | name2  | dbb1dd6ddb6b3c96ae72a44a8dd07edf | 631f74125db6e9217889851ce0a9c856 | 2025-06-20   | 2024-12-02
  34 | name | 51842df4de3b7da66c100a60a9547e7c | name2  | bc366a1cb380748d434f25bfafffa083 | dee3c9c4010b21fd95ab1c5daacb5a48 | 2025-03-12   | 2024-12-02
  35 | name | 0324c615f7a77fb86b4e076fcb39665a | name2  | 3c7c962560f46361f821c52d29578236 | 621014c24a5f6f0c646fba574709e166 | 2025-06-20   | 2025-03-12
  36 | name | 2d8e54322a472125ee2ad4f46c150e10 | name2  | 64757b9d6e384baaeaba0b9e7928126e | 54c64af831aee94d589849a0800e6214 | 2025-03-12   | 2025-03-12
  37 | name | 8e7bb26f72d23e2ec2094d2044198266 | name2  | e4209a959ca057e4681cbd3eb80cb306 | d9c9493f231826b1ccc5b2b2e6f63404 | 2025-03-12   | 2024-12-02

Дальше вывод вставлять не буду, инфо для наглядности :)
```

## Шаг 4. Выполняем бекап и восстанавливаемся с помощью утилит | pg_dump и pg_restore.

Для выполнения данного пункта берем ранее созданые 2 таблицы:

Фиксируем состояние до восстановления:

```
postgres=# \l
                                                        List of databases
    Name     |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype    | ICU Locale | ICU Rules |   Access privileges   
-------------+----------+----------+-----------------+-------------+-------------+------------+-----------+-----------------------
 demo        | postgres | UTF8     | libc            | ru_RU.UTF-8 | ru_RU.UTF-8 |            |           | 
 dvdrental   | postgres | UTF8     | libc            | ru_RU.UTF-8 | ru_RU.UTF-8 |            |           | 
 joins       | postgres | UTF8     | libc            | ru_RU.UTF-8 | ru_RU.UTF-8 |            |           | 
 postgres    | postgres | UTF8     | libc            | ru_RU.UTF-8 | ru_RU.UTF-8 |            |           | 
 task_backup | postgres | UTF8     | libc            | ru_RU.UTF-8 | ru_RU.UTF-8 |            |           | 
 template0   | postgres | UTF8     | libc            | ru_RU.UTF-8 | ru_RU.UTF-8 |            |           | =c/postgres          +
             |          |          |                 |             |             |            |           | postgres=CTc/postgres
 template1   | postgres | UTF8     | libc            | ru_RU.UTF-8 | ru_RU.UTF-8 |            |           | =c/postgres          +
             |          |          |                 |             |             |            |           | postgres=CTc/postgres
 trigger_db  | postgres | UTF8     | libc            | ru_RU.UTF-8 | ru_RU.UTF-8 |            |           | 
(8 rows)


task_backup=# \dt+ test_backup.*;
                                                List of relations
   Schema    |            Name            | Type  |  Owner   | Persistence | Access method | Size  | Description 
-------------+----------------------------+-------+----------+-------------+---------------+-------+-------------
 test_backup | document_template          | table | postgres | permanent   | heap          | 48 kB | 
 test_backup | document_template_for_copy | table | postgres | permanent   | heap          | 48 kB | 
(2 rows)


task_backup=# select count(*) from test_backup.document_template;
 count 
-------
   100
(1 row)

task_backup=# select count(*) from test_backup.document_template_for_copy;
 count 
-------
   100
(1 row)
```

Создаем бекап *pg_dump*-ом 2-ух таблиц в кастомном формате и смотрим содержимое получившегося результата:

```
postgres@studentPC:/etc/postgresql$ pg_dump -d task_backup --compress=9 --table=test_backup.document_template --table=test_backup.document_template_for_copy -Fc > /etc/postgresql/backup/backup_two_tables.gz
 
postgres@studentPC:/etc/postgresql$ ls -la /etc/postgresql/backup/
итого 44
drwxrwxr-x 2 postgres postgres  4096 июн 20 10:48 .
drwxr-xr-x 4 postgres postgres  4096 июн 20 10:24 ..
-rw-rw-r-- 1 postgres postgres 13492 июн 20 10:28 backup_copy_test_backup.document_template.sql
-rw-rw-r-- 1 postgres postgres 17407 июн 20 10:48 backup_two_tables.gz

postgres@studentPC:/etc/postgresql$ pg_restore --list /etc/postgresql/backup/backup_two_tables.gz
;
; Archive created at 2025-06-20 10:48:33 MSK
;     dbname: task_backup
;     TOC Entries: 8
;     Compression: gzip
;     Dump Version: 1.15-0
;     Format: CUSTOM
;     Integer: 4 bytes
;     Offset: 8 bytes
;     Dumped from database version: 16.9 (Ubuntu 16.9-0ubuntu0.24.04.1)
;     Dumped by pg_dump version: 16.9 (Ubuntu 16.9-0ubuntu0.24.04.1)
;
;
; Selected TOC Entries:
;
216; 1259 17280 TABLE test_backup document_template postgres
217; 1259 17288 TABLE test_backup document_template_for_copy postgres
3432; 0 17280 TABLE DATA test_backup document_template postgres
3433; 0 17288 TABLE DATA test_backup document_template_for_copy postgres
```

Приступаем к восстановлению, создаем новую БД и восстанавливаемся в нее:

```
postgres=# create database restore_task_backup;
CREATE DATABASE

postgres=# \c restore_task_backup;
You are now connected to database "restore_task_backup" as user "postgres".

restore_task_backup=# create schema test_backup;
CREATE SCHEMA

restore_task_backup=# \dt test_backup.*;
Did not find any relation named "test_backup.*".
```

Пробуем восстановится, по заданию восстанавливем только 2-ую таблицу:

```
postgres@studentPC:/etc/postgresql$ pg_restore -d restore_task_backup --table=document_template_for_copy /etc/postgresql/backup/backup_two_tables.gz;

restore_task_backup=# \dn 
         List of schemas
    Name     |       Owner       
-------------+-------------------
 public      | pg_database_owner
 test_backup | postgres
(2 rows)


restore_task_backup=# \dt+ test_backup.*;
                                                List of relations
   Schema    |            Name            | Type  |  Owner   | Persistence | Access method | Size  | Description 
-------------+----------------------------+-------+----------+-------------+---------------+-------+-------------
 test_backup | document_template_for_copy | table | postgres | permanent   | heap          | 48 kB | 
(1 row)

restore_task_backup=# select count(*) from test_backup.document_template_for_copy;
 count 
-------
   100
(1 row)

```

**В рамках данного домашенего задания удалось успешно воспользоваться 3-мя утилитами | *copy, pg_dump, pg_restore*;**