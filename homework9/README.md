Описание/Пошаговая инструкция выполнения домашнего задания:

  - создал и стартовал несколько кластеров (сделал на одной виртуальной машине):

    pg_lsclusters

    Ver Cluster Port Status Owner    Data directory               Log file

    14  main2   5433 online postgres /var/lib/postgresql/14/main2 /var/log/postgresql/postgresql-14-main2.log

    14  main3   5434 online postgres /var/lib/postgresql/14/main3 /var/log/postgresql/postgresql-14-main3.log

    14  main4   5435 online postgres /var/lib/postgresql/14/main4 /var/log/postgresql/postgresql-14-main4.log

  -

На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.
  - создал базу в кластере main2

    postgres=# create database test_otus;
    CREATE DATABASE

  - перешел в базу test_otus и создал две таблицы:

     postgres=# \c test_otus

     You are now connected to database "test_otus" as user "postgres".

     test_otus=# create table test(id int, name text);

     CREATE TABLE

     test_otus=# create table test2(id int, description text);

     CREATE TABLE
  - создал триггер на таблице test2 для предотвращения записи в эту таблицу:

    create function do_not_change()
    returns trigger
    as
    $$
    begin
      raise exception 'Cannot modify table.';
    end;
    $$
    language plpgsql;

    create trigger no_change_trigger
    before insert or update or delete on "test2"
    execute procedure do_not_change();

  - добавил записи в test. Попробовал добавить записи в test2:

    test_otus=# insert into test(id, name) values(1, 'test1'),(2, 'test2');

    INSERT 0 2

    test_otus=# select * from test;

      id | name

     ----+-------

       1 | test1

       2 | test2

     (2 rows)

     test_otus=# insert into test2(id, description) values(1, 'desc1'),(2, 'desc2');

     ERROR:  Cannot modify table.

     CONTEXT:  PL/pgSQL function do_not_change() line 3 at RAISE

  -


Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.
  - устанавливаем логическую репликацию и создаем публикацию таблицы test:

    test_otus=# ALTER SYSTEM SET wal_level = logical;

    ALTER SYSTEM

    test_otus=# CREATE PUBLICATION test_pub FOR TABLE test;

    WARNING:  wal_level is insufficient to publish logical changes

    HINT:  Set wal_level to logical before creating subscriptions.

    CREATE PUBLICATION

  - рестартуем кластер

     sudo pg_ctlcluster 14 main2 restart;

  -

На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.

  - создал базу в кластере main3

    postgres=# create database test_otus;
    CREATE DATABASE

  - перешел в базу test_otus и создал две таблицы:

     postgres=# \c test_otus

     You are now connected to database "test_otus" as user "postgres".

     test_otus=# create table test(id int, name text);

     CREATE TABLE

     test_otus=# create table test2(id int, description text);

     CREATE TABLE
  - создал триггер на таблице test для предотвращения записи в эту таблицу:

    create function do_not_change()
    returns trigger
    as
    $$
    begin
      raise exception 'Cannot modify table.';
    end;
    $$
    language plpgsql;

    create trigger no_change_trigger
    before insert or update or delete on "test"
    execute procedure do_not_change();

  - добавил записи в test2. Попробовал добавить записи в test:

    test_otus=# insert into test(id, name) values(1, 'test1'),(2, 'test2');

    ERROR:  Cannot modify table.

    CONTEXT:  PL/pgSQL function do_not_change() line 3 at RAISE

    test_otus=# insert into test2(id, description) values(1, 'desc1'),(2, 'desc2');

      INSERT 0 2

    test_otus=# select * from test2;

       id | description

      ----+-------------

        1 | desc1

        2 | desc2

      (2 rows)

  -

Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.

3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).


Небольшое описание, того, что получилось.


CREATE SUBSCRIPTION test2_sub
CONNECTION 'host=localhost port=5434 user=postgres password=123456 dbname=test_otus'
PUBLICATION test2_pub WITH (copy_data = true);
