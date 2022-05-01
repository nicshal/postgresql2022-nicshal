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
  - устанавливаем логическую репликацию, создаем публикацию таблицы test2, делаем подписку на test:

    test_otus=# ALTER SYSTEM SET wal_level = logical;

    ALTER SYSTEM

    test_otus=# CREATE PUBLICATION test2_pub FOR TABLE test2;

    WARNING:  wal_level is insufficient to publish logical changes

    HINT:  Set wal_level to logical before creating subscriptions.

    CREATE PUBLICATION

    test_otus=# CREATE SUBSCRIPTION test_sub

    test_otus-# CONNECTION 'host=localhost port=5433 user=postgres password=123456 dbname=test_otus'

    test_otus-# PUBLICATION test_pub WITH (copy_data = true);

    NOTICE:  created replication slot "test_sub" on publisher

    CREATE SUBSCRIPTION


  - рестартуем кластер

     sudo pg_ctlcluster 14 main3 restart;

  -

Теперь делаем подписку из кластера main2 на таблицу test2 в кластере main3:
  - делаем подписку:

    test_otus=# CREATE SUBSCRIPTION test2_sub

    CONNECTION 'host=localhost port=5434 user=postgres password=123456 dbname=test_otus'

    PUBLICATION test2_pub WITH (copy_data = true);

    NOTICE:  created replication slot "test2_sub" on publisher

    CREATE SUBSCRIPTION

Проверяем результаты:
  - из кластера main2 пытаемся обратиться к таблице test2:

    test_otus=# select * from test2;

      id | description

     ----+-------------

       1 | desc1

       2 | desc2

     (2 rows)

  - Данные видно. Логическая репликация настроена корректно


3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).
   создал базу в кластере main4

    postgres=# create database test_otus;
    CREATE DATABASE

  - перешел в базу test_otus и создал две таблицы:

     postgres=# \c test_otus

     You are now connected to database "test_otus" as user "postgres".

     test_otus=# create table test(id int, name text);

     CREATE TABLE

     test_otus=# create table test2(id int, description text);

     CREATE TABLE

  - создал триггер на таблицы test и test2 для предотвращения записи в эти таблицы:

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

    create trigger no_change_trigger2
    before insert or update or delete on "test2"
    execute procedure do_not_change();

  - делаем подписки на test и test2:

    test_otus=# CREATE SUBSCRIPTION test_sub_main4

    test_otus-# CONNECTION 'host=localhost port=5433 user=postgres password=123456 dbname=test_otus'

    test_otus-# PUBLICATION test_pub WITH (copy_data = true);

    NOTICE:  created replication slot "test_sub_main4" on publisher

    CREATE SUBSCRIPTION

    test_otus=# CREATE SUBSCRIPTION test2_sub_main4

    test_otus-# CONNECTION 'host=localhost port=5434 user=postgres password=123456 dbname=test_otus'

    test_otus-# PUBLICATION test2_pub WITH (copy_data = true);

    NOTICE:  created replication slot "test2_sub_main4" on publisher

    CREATE SUBSCRIPTION

  - проверяем результат:

    test_otus=# select * from test2;

       id | description

      ----+-------------

        1 | desc1

        2 | desc2

      (2 rows)

    test_otus=# select * from test;

       id | name

      ----+-------

        1 | test1

        2 | test2

      (2 rows)


  - Данные видно. Логическая репликация настроена корректно

  _

Небольшое описание, того, что получилось.
  - реализована логическая репликация между тремя кластерами.
  - При этом третий кластер main4 доступен только на чтение (некоторый аналог slave в схеме физической репликации)


Задание со звездочкой*
реализовать горячее реплицирование для высокой доступности на 4ВМ. Источником должна выступать ВМ №3. Написать с какими проблемами столкнулись.
  - проверяем настройки на кластере main4:

    test_otus=#

    test_otus=# show wal_level;

    wal_level

    -----------

     replica

    (1 row)

    test_otus=# show synchronous_commit;

    synchronous_commit

    --------------------

     on

    (1 row)

  - создаем еще один кластер:

    sudo pg_createcluster -d /var/lib/postgresql/14/main5 14 main5;

  - удалим файлы:

     sudo rm -rf /var/lib/postgresql/14/main5

  - сделаем бэкап с кластера main4:

     sudo -u postgres pg_basebackup -p 5435 -R -D /var/lib/postgresql/14/main5

  - стартуем кластер:

     sudo pg_ctlcluster 14 main5 start

  - проверяем настройки:

    postgres=# show hot_standby;

    hot_standby

    -------------

     on

    (1 row)

    postgres=# show hot_standby_feedback;

    hot_standby_feedback

    ----------------------

     off

    (1 row)


    postgres=# show max_standby_streaming_delay;

    max_standby_streaming_delay

    -----------------------------

    30s

    (1 row)

  - выставим меньшую задержку:

    postgres=# ALTER SYSTEM SET max_standby_streaming_delay=10;

    ALTER SYSTEM

  - рестартуем кластер:

    sudo pg_ctlcluster 14 main5 restart;

  - смотрим данные:

    postgres=# \c test_otus;

    You are now connected to database "test_otus" as user "postgres".

    test_otus=# select * from test;

       id | name

      ----+-------

        1 | test1

        2 | test2

      (2 rows)

    test_otus=# select * from test2;

       id | description

      ----+-------------

        1 | desc1

        2 | desc2

      (2 rows)

    - физическая репликация с кластера main4 на кластер main5 настроена успешно !!!







