Описание/Пошаговая инструкция выполнения домашнего задания:

Настройте выполнение контрольной точки раз в 30 секунд.
  + сделал:

    ALTER SYSTEM SET checkpoint_timeout TO '30s';

    sudo pg_ctlcluster 14 main restart;

10 минут c помощью утилиты pgbench подавайте нагрузку.
  + сделал

    журналы до запуска

        postgres=# SELECT * FROM pg_ls_waldir();

                  name           |   size   |      modification

        --------------------------+----------+------------------------

        000000010000000000000003 | 16777216 | 2022-04-10 14:15:50+00

        000000010000000000000002 | 16777216 | 2022-04-10 14:23:09+00


    sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres


    журналы после запуска:

        postgres=# SELECT * FROM pg_ls_waldir();

                  name           |   size   |      modification

        --------------------------+----------+------------------------

        000000010000000000000020 | 16777216 | 2022-04-10 14:45:42+00

        00000001000000000000001E | 16777216 | 2022-04-10 14:47:40+00

        000000010000000000000021 | 16777216 | 2022-04-10 14:46:11+00

        00000001000000000000001F | 16777216 | 2022-04-10 14:45:17+00


Измерьте, какой объем журнальных файлов был сгенерирован за это время. Оцените, какой объем приходится в среднем на одну контрольную точку.
  + измерил

    было сгенерировано 14 журнальных файлов общим объемом порядка 235 Mb

    в среднем на контрольную точку приходится около 12 MB


Проверьте данные статистики: все ли контрольные точки выполнялись точно по расписанию. Почему так произошло?
  + проверил

    контрольные точки выполнялись по расписанию

Сравните tps в синхронном/асинхронном режиме утилитой pgbench. Объясните полученный результат.
  + сравнил

    в синхронном режиме

    sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres;

    результат:

        pgbench (14.2 (Ubuntu 14.2-1.pgdg20.04+1+b1))

        starting vacuum...end.

        progress: 60.0 s, 507.5 tps, lat 15.736 ms stddev 20.882

        progress: 120.0 s, 610.4 tps, lat 13.085 ms stddev 18.495

        progress: 180.0 s, 615.0 tps, lat 12.984 ms stddev 10.847

        progress: 240.0 s, 565.1 tps, lat 14.134 ms stddev 13.042

        progress: 300.0 s, 635.1 tps, lat 12.573 ms stddev 10.378

        progress: 360.0 s, 603.6 tps, lat 13.231 ms stddev 11.525

        progress: 420.0 s, 574.5 tps, lat 13.902 ms stddev 11.667

        progress: 480.0 s, 567.9 tps, lat 14.066 ms stddev 11.585

        progress: 540.0 s, 610.7 tps, lat 13.078 ms stddev 10.766

        progress: 600.0 s, 542.1 tps, lat 14.734 ms stddev 12.024



    в асинхронном режиме

    ALTER SYSTEM SET synchronous_commit TO 'off';

    sudo pg_ctlcluster 14 main restart;

    sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres;

    результат:

        pgbench (14.2 (Ubuntu 14.2-1.pgdg20.04+1+b1))

        starting vacuum...end.

        progress: 60.0 s, 2794.6 tps, lat 2.829 ms stddev 7.281

        progress: 120.0 s, 2853.3 tps, lat 2.771 ms stddev 1.087

        progress: 180.0 s, 2812.4 tps, lat 2.811 ms stddev 7.339

        progress: 240.0 s, 2838.6 tps, lat 2.785 ms stddev 1.109

        progress: 300.0 s, 2816.6 tps, lat 2.807 ms stddev 1.119

        progress: 360.0 s, 2817.8 tps, lat 2.806 ms stddev 1.119

        progress: 420.0 s, 2822.2 tps, lat 2.802 ms stddev 1.109

        progress: 480.0 s, 2821.9 tps, lat 2.802 ms stddev 1.115

        progress: 540.0 s, 2813.0 tps, lat 2.810 ms stddev 1.124

        progress: 600.0 s, 2815.3 tps, lat 2.808 ms stddev 1.211


    tps выросла почти в пять раз. По всей видимости из-за резкого уменьшения операций ввода/вывода

Создайте новый кластер с включенной контрольной суммой страниц.
Создайте таблицу. Вставьте несколько значений. Выключите кластер.
Измените пару байт в таблице. Включите кластер и сделайте выборку из таблицы.
Что и почему произошло? как проигнорировать ошибку и продолжить работу?
  + сделал

    sudo pg_createcluster 14 test_2;

    sudo su - postgres -c '/usr/lib/postgresql/14/bin/pg_controldata -D "/var/lib/postgresql/14/test_2"' | grep state

        Database cluster state:               shut down

    sudo su - postgres -c '/usr/lib/postgresql/14/bin/pg_checksums --enable -D "/var/lib/postgresql/14/test_2"'

        Checksum operation completed

        Files scanned:  931

        Blocks scanned: 3216

        pg_checksums: syncing data directory

        pg_checksums: updating control file

        Checksums enabled in cluster

    sudo pg_ctlcluster 14 test_2 start;

    postgres=# show data_checksums;

        data_checksums

        ----------------

        on

    postgres=# create table test_test (id int, name text);

    CREATE TABLE

    postgres=# insert into test_test values(4, 'rrrrrrrrrrrrrrrrrr'), (2, 'ddddddddddddddddddddddddd'), (5, 'uuuuuuuuuuuuuuuuuuuuuuuu');

    INSERT 0 3

    postgres=# select * from test_test;

         id |           name

        ----+---------------------------

          4 | rrrrrrrrrrrrrrrrrr

          2 | ddddddddddddddddddddddddd

          5 | uuuuuuuuuuuuuuuuuuuuuuuu



    sudo pg_ctlcluster 14 test_2 stop;

    поменял пару байт в base/13760/16384

    sudo pg_ctlcluster 14 test_2 start;


    postgres=# select * from test_test;

        WARNING:  page verification failed, calculated checksum 37268 but expected 42638

        ERROR:  invalid page in block 0 of relation base/13760/16384

    ALTER SYSTEM SET ignore_checksum_failure TO 'on';

    sudo pg_ctlcluster 14 test_2 restart;


    postgres=# select * from test_test;

    WARNING:  page verification failed, calculated checksum 37268 but expected 42638

         id |           name

         ---+---------------------------

          4 | rrrrrrrrrrrrrrrrrr

          2 | ddddddddddddddddddddddddd

          5 | uuuuuuuuuuuuuuuuuuuuuuuu




