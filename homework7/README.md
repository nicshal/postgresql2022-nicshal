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

   -- во второй сессии:

      начинаем транзакцию:

      begin;

      update test_7 set name = 'name_7777' where id = 1;

      транзакция повисла из-за блокировки на обновляемой строке.

   ждем какое-то время

   - в первой сессии делаем commit;

   -- во второй сессии транзакция также завершилать

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

Воспроизведите взаимоблокировку трех транзакций. Можно ли разобраться в ситуации постфактум, изучая журнал сообщений?

Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?