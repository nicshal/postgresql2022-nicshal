Описание/Пошаговая инструкция выполнения домашнего задания:

1 создайте новый кластер PostgresSQL 13 (на выбор - GCE, CloudSQL)
 + сделал

2 зайдите в созданный кластер под пользователем postgres
  + зашел

3 создайте новую базу данных testdb
  + сделал:

    create database testdb;

4 зайдите в созданную базу данных под пользователем postgres
  + зашел:

    \c testdb

5 создайте новую схему testnm
  + сделал:

    create schema testnm;

6 создайте новую таблицу t1 с одной колонкой c1 типа integer
  + сделал:

    create table t1(c1 int);

7 вставьте строку со значением c1=1
  + вставил:

    insert into t1(c1) values(1);

8 создайте новую роль readonly
  + создал:

    create role readonly nologin;

9 дайте новой роли право на подключение к базе данных testdb
  + выдал

    grant connect on database testdb to readonly;

10 дайте новой роли право на использование схемы testnm
   + выдал:

     grant usage on schema testnm to readonly;

11 дайте новой роли право на select для всех таблиц схемы testnm
   + выдал

     grant select on all tables in schema testnm to readonly;


12 создайте пользователя testread с паролем test123
   + создал

     create user testread  with password 'test123';

13 дайте роль readonly пользователю testread
   + выдал

     grant readonly to testread;

14 зайдите под пользователем testread в базу данных testdb
   + зашел в новой сессии:

     sudo -u postgres psql -U testread  -h 51.250.100.207 -W -d testdb

15 сделайте select * from t1;
   + сделал

16 получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
   + не получилось

17 напишите что именно произошло в тексте домашнего задания
   + testdb=> select * from t1;

     ERROR:  permission denied for table t1

18 у вас есть идеи почему? ведь права то дали?
   + таблица t1 создана в схеме public. А права выдавались на схему testnm

19 посмотрите на список таблиц
   + testdb=> \dt

        List of relations

     Schema | Name | Type  |  Owner

    --------+------+-------+----------

     public | t1   | table | postgres

    (1 row)

20 подсказка в шпаргалке под пунктом 20
   + сверил версии

21 а почему так получилось с таблицей (если делали сами и без шпаргалки то может у вас все нормально)
   + в пути потска есть $user и public. Схем под созданных пользователей нет. При создании таблицы схема явно не указана. Поэтому создана в public

22 вернитесь в базу данных testdb под пользователем postgres
   + вернулся
     \c testdb postgres

23 удалите таблицу t1
   + удалил

     drop table t1;

24 создайте ее заново но уже с явным указанием имени схемы testnm
   + создал

   create table testnm.t1(c1 int);

25 вставьте строку со значением c1=1
   + вставил

     insert into testnm.t1(c1) values(1);

26 зайдите под пользователем testread в базу данных testdb
   + зашел

     \c testdb testread

27 сделайте select * from testnm.t1;
   + сделал

28 получилось?
   + не получилось

29 есть идеи почему? если нет - смотрите шпаргалку
   + таблица новая, только что создана. А GRANT на схему выдали раньше, чем создали таблицу. И этот GRANT не действует на вновь созданные таблицы

30 как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
   + сделал

     \c testdb postgres;

     alter default privileges in schema testnm grant select on tables to readonly;

31 сделайте select * from testnm.t1;
   + сделал

     \c testdb testread;

     select * from testnm.t1;

32 получилось?
   + не получилось

33 есть идеи почему? если нет - смотрите шпаргалку
   + alter default privileges действует для новых таблиц. GRANT вызывался до создания таблицы testnm.t1. Поэтомы текущая версия таблицы не попала под раздачу доступов

   \c testdb postgres;

   grant select on all tables in schema testnm to readonly;

   \c testdb testread;

34 сделайте select * from testnm.t1;
   + сделал

35 получилось?
   + получилось

36 ура!
   + ура

37 теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
   + попробовал - получилось

38 а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?
   + создали таблицу в схеме public (из пути поиска). Все роли по умолчанию наследуют от роли public. А для роли public разрешено создание объектов в схеме public

39 есть идеи как убрать эти права? если нет - смотрите шпаргалку
   + нужно ограничить роль public

   \c testdb postgres;

   revoke create on schema public from public;

   revoke all on database testdb from public;

   \c testdb testread;

40 если вы справились сами то расскажите что сделали и почему, если смотрели шпаргалку - объясните что сделали и почему выполнив указанные в ней команды
   + практически все сделал сам. Посмотрел в шпаргалку один раз (по alter default privileges)

41 теперь попробуйте выполнить команду create table t3(c1 integer); insert into t2 values (2);
   + выполнил

42 расскажите что получилось и почему
   + не получилось

   testdb=> create table t3(c1 integer); insert into t3 values (2);

   ERROR:  permission denied for schema public

   LINE 1: create table t3(c1 integer);
                     ^
   ERROR:  relation "t3" does not exist

   LINE 1: insert into t3 values (2);
                    ^
