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

   postgres=# \d

              List of relations

   Schema |       Name       | Type  |  Owner

  --------+------------------+-------+----------

   public | pgbench_accounts | table | postgres

   public | pgbench_branches | table | postgres

   public | pgbench_history  | table | postgres

   public | pgbench_tellers  | table | postgres

  (4 rows)

запустить pgbench -c8 -P 60 -T 3600 -U postgres postgres

дать отработать до конца

зафиксировать среднее значение tps в последней ⅙ части работы

а дальше настроить autovacuum максимально эффективно

так чтобы получить максимально ровное значение tps на горизонте часа