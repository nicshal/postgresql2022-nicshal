Описание/Пошаговая инструкция выполнения домашнего задания:

развернуть виртуальную машину любым удобным способом
  + сделал

поставить на неё PostgreSQL 14 из пакетов собираемых postgres.org
  + сделал

настроить кластер PostgreSQL 14 на максимальную производительность не
обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины
  + для начала воспользуюсь рекомендациями с https://pgtune.leopard.in.ua/#/ для текущей конфигурации

      - DB Version: 14
      - OS Type: linux
      - DB Type: oltp
      - Total Memory (RAM): 4 GB
      - CPUs num: 2
      - Connections num: 100
      - Data Storage: ssd

      Рекомендуемые     -------------------------- По умолчанию

      max_connections = 100

      shared_buffers = 1GB  --------------------------(128MB)

      effective_cache_size = 3GB                 (4GB)

      maintenance_work_mem = 256MB               (64MB)

      checkpoint_completion_target = 0.9

      wal_buffers = 16MB                         (4MB)

      default_statistics_target = 100

      random_page_cost = 1.1                     (4)

      effective_io_concurrency = 200             (1)

      work_mem = 10485kB                         (4MB)

      min_wal_size = 2GB                         (80MB)

      max_wal_size = 8GB                         (1GB)

      max_worker_processes = 2                   (8)

      max_parallel_workers_per_gather = 1        (2)

      max_parallel_workers = 2                   (8)

      max_parallel_maintenance_workers = 1       (2)

нагрузить кластер через утилиту
https://github.com/Percona-Lab/sysbench-tpcc (требует установки
https://github.com/akopytov/sysbench) или через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
  + для тестирования буду использовать pgbench

    - для начала запускаем pgbench с настройками по умолчанию:

      sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres;

      pgbench (14.2 (Ubuntu 14.2-1.pgdg20.04+1+b1))

      starting vacuum...end.

      progress: 60.0 s, 619.5 tps, lat 12.886 ms stddev 10.656

      progress: 120.0 s, 696.1 tps, lat 11.471 ms stddev 8.955

      progress: 180.0 s, 636.5 tps, lat 12.544 ms stddev 10.480

      progress: 240.0 s, 641.1 tps, lat 12.458 ms stddev 10.610

      progress: 300.0 s, 503.1 tps, lat 15.880 ms stddev 13.916

      progress: 360.0 s, 664.2 tps, lat 12.022 ms stddev 23.383

      progress: 420.0 s, 631.8 tps, lat 12.641 ms stddev 18.732

      progress: 480.0 s, 568.4 tps, lat 14.050 ms stddev 18.731

      progress: 540.0 s, 508.1 tps, lat 15.727 ms stddev 21.764

      progress: 600.0 s, 665.7 tps, lat 11.995 ms stddev 17.978

      transaction type: <builtin: TPC-B (sort of)>

      scaling factor: 1

      query mode: simple

      number of clients: 8

      number of threads: 1

      duration: 600 s

      number of transactions actually processed: 368078

      latency average = 13.019 ms

      latency stddev = 16.239 ms

      initial connection time = 14.459 ms

      tps = 613.459155 (without initial connection time)



    - внес изменения по рекомендациям с leopard:

      ALTER SYSTEM SET shared_buffers TO '1GB';

      ALTER SYSTEM SET effective_cache_size TO '3GB';

      ALTER SYSTEM SET maintenance_work_mem TO '256MB';

      ALTER SYSTEM SET wal_buffers TO '16MB';

      ALTER SYSTEM SET random_page_cost TO '1.1';

      ALTER SYSTEM SET effective_io_concurrency TO '200';

      ALTER SYSTEM SET work_mem TO '10485kB';

      ALTER SYSTEM SET min_wal_size TO '2GB';

      ALTER SYSTEM SET max_wal_size TO '8GB';

      ALTER SYSTEM SET max_worker_processes TO '2';

      ALTER SYSTEM SET max_parallel_workers_per_gather TO '1';

      ALTER SYSTEM SET max_parallel_workers TO '2';

      ALTER SYSTEM SET max_parallel_maintenance_workers TO '1';

      SELECT pg_reload_conf();

    - запускаем pgbench

      sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres;

        pgbench (14.2 (Ubuntu 14.2-1.pgdg20.04+1+b1))

        starting vacuum...end.

        progress: 60.0 s, 618.1 tps, lat 12.917 ms stddev 10.630

        progress: 120.0 s, 651.4 tps, lat 12.259 ms stddev 10.548

        progress: 180.0 s, 594.4 tps, lat 13.439 ms stddev 11.355

        progress: 240.0 s, 550.3 tps, lat 14.515 ms stddev 12.698

        progress: 300.0 s, 708.7 tps, lat 11.267 ms stddev 8.495

        progress: 360.0 s, 662.1 tps, lat 12.059 ms stddev 21.801

        progress: 420.0 s, 575.5 tps, lat 13.877 ms stddev 19.415

        progress: 480.0 s, 660.8 tps, lat 12.086 ms stddev 18.271

        progress: 540.0 s, 662.2 tps, lat 12.056 ms stddev 9.695

        progress: 600.0 s, 699.7 tps, lat 11.414 ms stddev 9.023

        transaction type: <builtin: TPC-B (sort of)>

        scaling factor: 1

        query mode: simple

        number of clients: 8

        number of threads: 1

        duration: 600 s

        number of transactions actually processed: 382990

        latency average = 12.511 ms

        latency stddev = 13.918 ms

        initial connection time = 14.093 ms

        tps = 638.309664 (without initial connection time)

    ВЫВОД - РЕКОМЕНДАЦИИ ПОМОГЛИ НЕЗНАЧИТЕЛЬНО (~ 5%) УВЕЛИЧИТЬ ПРОИЗВОДИТЕЛЬНОСТЬ.
    Поигрался немного с вышеперечисленными настройками. Максимально удалось добиться tps ~= 650.
    То есть удалось добиться повышения на 5 - 7 % по сравнению с настройками по умолчанию


    - внесем еще некоторые изменения

      ALTER SYSTEM SET synchronous_commit TO 'off';

      SELECT pg_reload_conf();

    - запускаем pgbench

      sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres;

      pgbench (14.2 (Ubuntu 14.2-1.pgdg20.04+1+b1))

      starting vacuum...end.

      progress: 60.0 s, 2810.6 tps, lat 2.813 ms stddev 1.101

      progress: 120.0 s, 2816.3 tps, lat 2.808 ms stddev 1.101

      progress: 180.0 s, 2817.3 tps, lat 2.806 ms stddev 2.417

      progress: 240.0 s, 2801.1 tps, lat 2.823 ms stddev 4.651

      progress: 300.0 s, 2828.0 tps, lat 2.794 ms stddev 2.198

      progress: 360.0 s, 2854.8 tps, lat 2.768 ms stddev 1.121

      progress: 420.0 s, 2836.9 tps, lat 2.786 ms stddev 1.121

      progress: 480.0 s, 2824.7 tps, lat 2.798 ms stddev 1.135

      progress: 540.0 s, 2811.5 tps, lat 2.813 ms stddev 1.182

      progress: 600.0 s, 2796.6 tps, lat 2.827 ms stddev 1.112

      transaction type: <builtin: TPC-B (sort of)>

      scaling factor: 1

      query mode: simple

      number of clients: 8

      number of threads: 1

      duration: 600 s

      number of transactions actually processed: 1691868

      latency average = 2.804 ms

      latency stddev = 2.026 ms

      initial connection time = 14.403 ms

      tps = 2819.770145 (without initial connection time)

    - ВЫВОД - ПОЛУЧИЛИ ПОВЫШЕНИЕ ПРОИЗВОДИТЕЛЬНОСТИ БОЛЬШЕ ЧЕМ В ЧЕТЫРЕ РАЗА


написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему
  + ВЫВОД - БОЛЬШЕ ВСЕГО ПОВЫСИТЬ ПРОИЗВОДИТЕЛЬНОСТЬ ПОМАГАЕТ ПЕРЕВОД СИНХРОННЫХ ОПЕРАЦИЙ В АСИНХРОННЫЙ РЕЖИМ.
    НО С ОТКЛЮЧЕНИЕМ СИНХРОННЫХ РЕЖИМОВ ПОВЫШАЕТСЯ РИСК ПОТЕРИ ДАННЫХ