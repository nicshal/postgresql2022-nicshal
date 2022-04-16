Описание/Пошаговая инструкция выполнения домашнего задания:

Настройте сервер так, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд.
Воспроизведите ситуацию, при которой в журнале появятся такие сообщения.
 + сделал

   ALTER SYSTEM SET deadlock_timeout = 200;

   ALTER SYSTEM SET log_lock_waits = on;

   SELECT pg_reload_conf();

   - в первой сессии:

     create table test_7(id int, name text);

     insert into test_7(id, name) values(1, 'test_7');

     начинаем транзакцию:

     begin;

     update test_7 set name = 'name_77' where id = 1;

   - во второй сессии:

      начинаем транзакцию:

      begin;

      update test_7 set name = 'name_7777' where id = 1;

      обработка повисла из-за блокировки на обновляемой строке.

   ждем какое-то время

   - в первой сессии делаем commit;

   - во второй сессии транзакция также завершилать

      делаем commit во второй сессии

   смотрим журнал:

   sudo tail -n 10 /var/log/postgresql/postgresql-14-main.log

   2022-04-16 18:01:09.132 UTC [586] LOG:  parameter "deadlock_timeout" changed to "200"

   2022-04-16 18:08:25.562 UTC [756] postgres@postgres ERROR:  syntax error at or near "test_7" at character 8

   2022-04-16 18:08:25.562 UTC [756] postgres@postgres STATEMENT:  insert test_7(id, name) values(1, 'test_7');

   2022-04-16 18:13:23.224 UTC [799] postgres@postgres LOG:  process 799 still waiting for ShareLock on transaction 2043139 after 200.055 ms

   2022-04-16 18:13:23.224 UTC [799] postgres@postgres DETAIL:  Process holding the lock: 756. Wait queue: 799.

   2022-04-16 18:13:23.224 UTC [799] postgres@postgres CONTEXT:  while updating tuple (0,1) in relation "test_7"

   2022-04-16 18:13:23.224 UTC [799] postgres@postgres STATEMENT:  update test_7 set name = 'name_7777' where id = 1;

   2022-04-16 18:15:40.577 UTC [799] postgres@postgres LOG:  process 799 acquired ShareLock on transaction 2043139 after 137553.911 ms

   2022-04-16 18:15:40.577 UTC [799] postgres@postgres CONTEXT:  while updating tuple (0,1) in relation "test_7"

   2022-04-16 18:15:40.577 UTC [799] postgres@postgres STATEMENT:  update test_7 set name = 'name_7777' where id = 1;

   В ЖУРНАЛЕ ВИДИМ СООБЩЕНИЕ О ВОЗНИКШЕЙ БЛОКИРОВКЕ !!!



Смоделируйте ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах.
Изучите возникшие блокировки в представлении pg_locks и убедитесь, что все они понятны.
Пришлите список блокировок и объясните, что значит каждая.
  + сделал

    - в первой сессии:

      begin;

      update test_7 set name = 'name_88' where id = 1;  (pid 756)

    - во второй сессии:

      begin;

      update test_7 set name = 'name_8888' where id = 1;

      обработка повисла из-за блокировки на обновляемой строке (pid 799)

    - в третьей сессии:

      begin;

      update test_7 set name = 'name_888888' where id = 1;

      обработка повисла из-за блокировки на обновляемой строке (pid 1093).

    смотрим возникшие блокировки:

    SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for FROM pg_locks WHERE relation = 'test_7'::regclass;

    locktype |       mode       | granted | pid  | wait_for

   ----------+------------------+---------+------+----------

    relation | RowExclusiveLock | t       | 1093 | {799}

    relation | RowExclusiveLock | t       |  799 | {756}

    relation | RowExclusiveLock | t       |  756 | {}

    tuple    | ExclusiveLock    | f       | 1093 | {799}

    tuple    | ExclusiveLock    | t       |  799 | {756}


    Можно видеть, что:
     - транзакция в процессе 756 выполнилась и ничего не ждет
     - транзакция в процессе 799 ждет завершения транзакции в процессе 756
     - транзакция в процессе 1093 ждет завершения транзакции в процессе 7799

   - делаем commit в первой сессии

     транзакция в процессе 799 отвисла и продолжила выполнение

     SELECT locktype, mode, granted, pid, pg_blocking_pids(pid) AS wait_for FROM pg_locks WHERE relation = 'test_7'::regclass;

     locktype |       mode       | granted | pid  | wait_for

    ----------+------------------+---------+------+----------

     relation | RowExclusiveLock | t       | 1093 | {799}

     relation | RowExclusiveLock | t       |  799 | {}

  - делаем commit во второй сессии.

    транзакция в процессе 1093 отвисла и продолжила выполнение



Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?
  + сделал

    - в первой сессии:

      begin;

      update test_7 set name = name || 'xxx'  where id = 1;

      - во второй сессии:

        begin;

        update test_7 set name = name || 'xxx'  where id = 2;

       - в третьей сессии:

         begin;

         update test_7 set name = name || 'xxx'  where id = 3;

    - в первой сессии:

      update test_7 set name = name || 'yyy'  where id = 2;

      - во второй сессии:

        update test_7 set name = name || 'yyy'  where id = 3;

       - в третьей сессии:

         update test_7 set name = name || 'yyy'  where id = 1;

    В СЕССИИ ПОЛУЧИЛИ СООБЩЕНИЕ:

    ERROR:  deadlock detected

    DETAIL:  Process 1093 waits for ShareLock on transaction 2043156; blocked by process 756.

    Process 756 waits for ShareLock on transaction 2043157; blocked by process 799.

    Process 799 waits for ShareLock on transaction 2043158; blocked by process 1093.

    HINT:  See server log for query details.

    CONTEXT:  while updating tuple (0,21) in relation "test_7"

    Делаем commit во всех сессиях

    Смотрим логи sudo tail -n 10 /var/log/postgresql/postgresql-14-main.log

    ВИДИМ:

    2022-04-16 20:06:03.854 UTC [1093] postgres@postgres ERROR:  deadlock detected

    2022-04-16 20:06:03.854 UTC [1093] postgres@postgres DETAIL:  Process 1093 waits for ShareLock on transaction 2043156; blocked by process 756.

	         Process 756 waits for ShareLock on transaction 2043157; blocked by process 799.

	         Process 799 waits for ShareLock on transaction 2043158; blocked by process 1093.

	         Process 1093: update test_7 set name = name || 'yyy'  where id = 1;

	         Process 756: update test_7 set name = name || 'yyy'  where id = 2;

	         Process 799: update test_7 set name = name || 'yyy'  where id = 3;

    2022-04-16 20:06:03.854 UTC [1093] postgres@postgres HINT:  See server log for query details.

    2022-04-16 20:06:03.854 UTC [1093] postgres@postgres CONTEXT:  while updating tuple (0,21) in relation "test_7"


    В ЖУРНАЛЕ ПРЕДСТАВЛЕНА ДОСТАТОЧНАЯ ИНФОРМАЦИЯ ДЛЯ РАЗБОРА ВОЗНИКШЕЙ СИТУАЦИИ


Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
  + сделал

    Операция UPDATE проходит не единовременно для всех строк. Строки блокируются по мере их обновления (на таблице висит блокировка RowExclusiveLock, полностью таблица не блокирована)

    Если для команд на полный UPDATE одно и той же таблицы в разных сессиях будут построены разные планы выполнения, то теоретическт взаимоблокировка возможна


