Описание/Пошаговая инструкция выполнения домашнего задания:

создать GCE инстанс типа e2-medium и standard disk 10GB
  + создал

установить на него PostgreSQL 13 с дефолтными настройками
  + установил

применить параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
  + применил новые настройки:

    ALTER SYSTEM SET max_connections TO '40';

    ALTER SYSTEM SET shared_buffers TO '1GB';

    ALTER SYSTEM SET effective_cache_size TO '3GB';

    ALTER SYSTEM SET maintenance_work_mem TO '512MB';

    ALTER SYSTEM SET default_statistics_target TO '500';

    ALTER SYSTEM SET effective_io_concurrency TO '2';

    ALTER SYSTEM SET work_mem TO '6553kB';

    ALTER SYSTEM SET min_wal_size TO '4GB';

    ALTER SYSTEM SET max_wal_size TO '16GB';

    sudo pg_ctlcluster 14 main restart

выполнить pgbench -i postgres
  + выполнил

     sudo -u postgres pgbench -i postgres

     postgres=# \d

     Schema |       Name       | Type  |  Owner

    --------+------------------+-------+----------

     public | pgbench_accounts | table | postgres

     public | pgbench_branches | table | postgres

     public | pgbench_history  | table | postgres

     public | pgbench_tellers  | table | postgres

     (4 rows)

запустить pgbench -c8 -P 60 -T 3600 -U postgres postgres
  + запустил

    sudo -u postgres pgbench -c8 -P 60 -T 3600 -U postgres postgres

дать отработать до конца
  + дождался результата:

        pgbench (14.2 (Ubuntu 14.2-1.pgdg20.04+1))

        starting vacuum...end.

        progress: 60.0 s, 676.7 tps, lat 11.816 ms stddev 8.149

        progress: 120.0 s, 550.0 tps, lat 14.548 ms stddev 10.741

        progress: 180.0 s, 588.8 tps, lat 13.588 ms stddev 9.227

        progress: 240.0 s, 567.8 tps, lat 14.087 ms stddev 9.989

        progress: 300.0 s, 627.6 tps, lat 12.742 ms stddev 9.200

        progress: 360.0 s, 486.2 tps, lat 16.460 ms stddev 12.649

        progress: 420.0 s, 612.1 tps, lat 13.063 ms stddev 9.127

        progress: 480.0 s, 595.6 tps, lat 13.436 ms stddev 9.442

        progress: 540.0 s, 580.0 tps, lat 13.794 ms stddev 10.149

        progress: 600.0 s, 567.1 tps, lat 14.106 ms stddev 9.958

        progress: 660.0 s, 632.4 tps, lat 12.644 ms stddev 8.595

        progress: 720.0 s, 559.4 tps, lat 14.304 ms stddev 10.234

        progress: 780.0 s, 651.6 tps, lat 12.280 ms stddev 7.838

        progress: 840.0 s, 535.9 tps, lat 14.923 ms stddev 11.394

        progress: 900.0 s, 643.7 tps, lat 12.429 ms stddev 7.934

        progress: 960.0 s, 585.9 tps, lat 13.653 ms stddev 9.863

        progress: 1020.0 s, 570.3 tps, lat 14.027 ms stddev 10.002

        progress: 1080.0 s, 532.7 tps, lat 15.018 ms stddev 11.394

        progress: 1140.0 s, 594.7 tps, lat 13.453 ms stddev 9.746

        progress: 1200.0 s, 618.2 tps, lat 12.941 ms stddev 8.992

        progress: 1260.0 s, 597.5 tps, lat 13.387 ms stddev 9.119

        progress: 1320.0 s, 558.7 tps, lat 14.319 ms stddev 9.505

        progress: 1380.0 s, 617.1 tps, lat 12.963 ms stddev 8.581

        progress: 1440.0 s, 623.2 tps, lat 12.838 ms stddev 8.366

        progress: 1500.0 s, 645.7 tps, lat 12.386 ms stddev 8.085

        progress: 1560.0 s, 484.6 tps, lat 16.507 ms stddev 11.805

        progress: 1620.0 s, 611.7 tps, lat 13.079 ms stddev 8.686

        progress: 1680.0 s, 569.6 tps, lat 14.045 ms stddev 9.454

        progress: 1740.0 s, 610.1 tps, lat 13.111 ms stddev 9.076

        progress: 1800.0 s, 552.8 tps, lat 14.471 ms stddev 10.324

        progress: 1860.0 s, 636.3 tps, lat 12.570 ms stddev 8.212

        progress: 1920.0 s, 624.4 tps, lat 12.809 ms stddev 8.492

        progress: 1980.0 s, 602.2 tps, lat 13.284 ms stddev 8.675

        progress: 2040.0 s, 584.1 tps, lat 13.696 ms stddev 9.719

        progress: 2100.0 s, 655.8 tps, lat 12.201 ms stddev 7.971

        progress: 2160.0 s, 567.5 tps, lat 14.093 ms stddev 10.315

        progress: 2220.0 s, 582.6 tps, lat 13.732 ms stddev 9.645

        progress: 2280.0 s, 564.9 tps, lat 14.162 ms stddev 9.696

        progress: 2340.0 s, 484.0 tps, lat 16.527 ms stddev 13.518

        progress: 2400.0 s, 651.3 tps, lat 12.283 ms stddev 7.719

        progress: 2460.0 s, 628.8 tps, lat 12.722 ms stddev 8.825

        progress: 2520.0 s, 578.3 tps, lat 13.830 ms stddev 9.519

        progress: 2580.0 s, 581.3 tps, lat 13.764 ms stddev 9.346

        progress: 2640.0 s, 582.8 tps, lat 13.725 ms stddev 9.593

        progress: 2700.0 s, 615.4 tps, lat 13.000 ms stddev 8.620

        progress: 2760.0 s, 651.8 tps, lat 12.274 ms stddev 7.524

        progress: 2820.0 s, 593.7 tps, lat 13.475 ms stddev 9.800

        progress: 2880.0 s, 610.4 tps, lat 13.106 ms stddev 8.926

        progress: 2940.0 s, 628.4 tps, lat 12.730 ms stddev 8.308

        progress: 3000.0 s, 613.4 tps, lat 13.041 ms stddev 8.697

        progress: 3060.0 s, 549.6 tps, lat 14.556 ms stddev 11.142

        progress: 3120.0 s, 568.4 tps, lat 14.073 ms stddev 9.617

        progress: 3180.0 s, 645.3 tps, lat 12.397 ms stddev 7.808

        progress: 3240.0 s, 572.7 tps, lat 13.968 ms stddev 9.502

        progress: 3300.0 s, 576.6 tps, lat 13.873 ms stddev 9.903

        progress: 3360.0 s, 642.6 tps, lat 12.450 ms stddev 8.037

        progress: 3420.0 s, 600.0 tps, lat 13.333 ms stddev 8.707

        progress: 3480.0 s, 605.6 tps, lat 13.209 ms stddev 9.010

        progress: 3540.0 s, 578.0 tps, lat 13.839 ms stddev 9.538

        progress: 3600.0 s, 559.9 tps, lat 14.288 ms stddev 10.471

        transaction type: <builtin: TPC-B (sort of)>

        scaling factor: 1

        query mode: simple

        number of clients: 8

        number of threads: 1

        duration: 3600 s

        number of transactions actually processed: 2135034

        latency average = 13.489 ms

        latency stddev = 9.449 ms

        initial connection time = 14.822 ms

        tps = 593.063869 (without initial connection time)

зафиксировать среднее значение tps в последней ⅙ части работы
  + зафиксировал - 589.87

а дальше настроить autovacuum максимально эффективно

так чтобы получить максимально ровное значение tps на горизонте часа