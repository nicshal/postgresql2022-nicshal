Описание/Пошаговая инструкция выполнения домашнего задания:

  - для выполнения задания скопировал на сервер архив с demo-базой (https://postgrespro.com/education/demo-medium-en.zip)

1 вариант:

Создать индексы на БД, которые ускорят доступ к данным.

   - создаем индекс на таблицу bookings.tickets - для ускорения поиска по имени пассажира

     - пробуем запрос без индекса

       explain analyse

       select * from bookings.tickets where passenger_name = 'DAMIR TIMOFEEV';

       - результат:
         Gather  (cost=1000.00..19215.58 rows=75 width=104) (actual time=0.244..88.647 rows=4 loops=1)

           Workers Planned: 2

           Workers Launched: 2

           ->  Parallel Seq Scan on tickets  (cost=0.00..18208.08 rows=31 width=104) (actual time=38.937..83.021 rows=1 loops=3)

               Filter: (passenger_name = 'DAMIR TIMOFEEV'::text)

               Rows Removed by Filter: 276356

         Planning Time: 0.064 ms

         Execution Time: 88.665 ms

     - создаем индекс

       create index bookings_passenger_name_idx on bookings.tickets (passenger_name);

     - пробуем запрос с индексом - получаем значительное уменьшение времени выполнения запроса

       explain analyse

       select * from bookings.tickets where passenger_name = 'DAMIR TIMOFEEV';

       - результат:
         Bitmap Heap Scan on tickets  (cost=5.01..289.41 rows=75 width=104) (actual time=0.040..0.065 rows=4 loops=1)

           Recheck Cond: (passenger_name = 'DAMIR TIMOFEEV'::text)

           Heap Blocks: exact=4

           ->  Bitmap Index Scan on bookings_passenger_name_idx  (cost=0.00..4.99 rows=75 width=0) (actual time=0.035..0.036 rows=4 loops=1)

               Index Cond: (passenger_name = 'DAMIR TIMOFEEV'::text)

         Planning Time: 0.217 ms

         Execution Time: 0.081 ms

  - создаем индекс для полнотекстового поиска

    - запрос без индекса
      explain analyse

      select * from bookings.tickets where passenger_name @@ 'TIMOFEEV';

    - результат
      Gather  (cost=1000.00..105472.93 rows=7669 width=104) (actual time=2.902..2780.377 rows=3065 loops=1)

        Workers Planned: 2

        Workers Launched: 2

        ->  Parallel Seq Scan on tickets  (cost=0.00..103706.03 rows=3195 width=104) (actual time=6.982..2754.754 rows=1022 loops=3)

            Filter: (passenger_name @@ 'TIMOFEEV'::text)

            Rows Removed by Filter: 275335

      Planning Time: 0.829 ms

      JIT:

        Functions: 6

        Options: Inlining false, Optimization false, Expressions true, Deforming true

        Timing: Generation 1.288 ms, Inlining 0.000 ms, Optimization 1.177 ms, Emission 17.209 ms, Total 19.674 ms

      Execution Time: 2781.034 ms

    - еще один запрос без индекса
      explain analyse

      select * from bookings.tickets where passenger_name like '%TIMOFEEV';

    - результат

      Gather  (cost=1000.00..19974.98 rows=7669 width=104) (actual time=0.234..134.319 rows=3065 loops=1)

        Workers Planned: 2

        Workers Launched: 2

          ->  Parallel Seq Scan on tickets  (cost=0.00..18208.08 rows=3195 width=104) (actual time=0.143..113.523 rows=1022 loops=3)

              Filter: (passenger_name ~~ '%TIMOFEEV'::text)

              Rows Removed by Filter: 275335

      Planning Time: 0.081 ms

      Execution Time: 134.581 ms

    - создаем gin-индекс

      - подключаем расширения

        CREATE EXTENSION pg_trgm;

        CREATE EXTENSION btree_gin;

      - создаем новое поле в таблице, заполняем его значениями, делаем gin-индекс по новому полю

        alter table bookings.tickets add column name_tsv tsvector;

        update bookings.tickets set name_tsv = to_tsvector(passenger_name);

        create index bookings_passenger_name_git_idx on bookings.tickets using gin(name_tsv);

      - повторяем запрос
        explain analyse

        select * from bookings.tickets where name_tsv @@ 'TIMOFEEV';

      - результат - видим использование индекса и значительное уменьшение времени выполнения запроса

        Bitmap Heap Scan on tickets  (cost=16.43..230.23 rows=55 width=140) (actual time=0.011..0.011 rows=0 loops=1)

          Recheck Cond: (name_tsv @@ '''TIMOFEEV'''::tsquery)

          ->  Bitmap Index Scan on bookings_passenger_name_git_idx  (cost=0.00..16.41 rows=55 width=0) (actual time=0.010..0.010 rows=0 loops=1)

                Index Cond: (name_tsv @@ '''TIMOFEEV'''::tsquery)

        Planning Time: 0.091 ms

        Execution Time: 0.032 ms

  - создаем индекс на часть таблицы

    - делаем запрос (индекса нет)
      explain analyse

      select flight_no from bookings.flights where status = 'On Time';

    - результат

        Seq Scan on flights  (cost=0.00..1612.80 rows=530 width=7) (actual time=0.008..6.663 rows=518 loops=1)

          Filter: ((status)::text = 'On Time'::text)

          Rows Removed by Filter: 65146

        Planning Time: 0.085 ms

        Execution Time: 6.699 ms

    - делаем индекс create index flights_flight_no_idx on bookings.flights(flight_no) where status = 'On Time';

    - повторяем запрос
      explain analyse

      select flight_no from bookings.flights where status = 'On Time';

    - результат - видим использование индекса

        Index Only Scan using flights_flight_no_idx on flights  (cost=0.28..24.16 rows=530 width=7) (actual time=0.027..0.070 rows=518 loops=1)

          Heap Fetches: 0

        Planning Time: 0.201 ms

        Execution Time: 0.096 ms

    - повторяем запрос (подбираем условин мимо индекса)
      explain analyse

      select flight_no from bookings.flights where status = 'Arrived';

    - результат - индекс не используется

      Seq Scan on flights  (cost=0.00..1612.80 rows=49364 width=7) (actual time=0.011..8.343 rows=49235 loops=1)

        Filter: ((status)::text = 'Arrived'::text)

        Rows Removed by Filter: 16429

      Planning Time: 0.082 ms

      Execution Time: 9.869 ms

  - создаем индекс на несколько полей

    - запрос без индекса
      explain analyse

      select flight_no from bookings.flights where departure_airport = 'DME' and arrival_airport = 'LED';

    - результат

      Seq Scan on flights  (cost=0.00..1776.96 rows=378 width=7) (actual time=0.007..6.216 rows=484 loops=1)

        Filter: ((departure_airport = 'DME'::bpchar) AND (arrival_airport = 'LED'::bpchar))

        Rows Removed by Filter: 65180

      Planning Time: 0.101 ms

      Execution Time: 6.245 ms

    - делаем индекс create index flights_airport_idx on bookings.flights(departure_airport, arrival_airport);

    - повторяем запрос
      explain analyse

      select flight_no from bookings.flights where departure_airport = 'DME' and arrival_airport = 'LED';

    - результат - видим использование индекса

      Bitmap Heap Scan on flights  (cost=8.17..667.23 rows=378 width=7) (actual time=0.042..0.096 rows=484 loops=1)

        Recheck Cond: ((departure_airport = 'DME'::bpchar) AND (arrival_airport = 'LED'::bpchar))

        Heap Blocks: exact=7

        ->  Bitmap Index Scan on flights_airport_idx  (cost=0.00..8.07 rows=378 width=0) (actual time=0.034..0.034 rows=484 loops=1)

            Index Cond: ((departure_airport = 'DME'::bpchar) AND (arrival_airport = 'LED'::bpchar))

      Planning Time: 0.214 ms

      Execution Time: 0.125 ms


  -
  -


2 вариант: В результате выполнения ДЗ вы научитесь пользоваться различными вариантами соединения таблиц.

  - для проведения экспериментов предварительно удалил ограничение foreign key на таблицах bookings.tickets и bookings.ticket_flights  . Также удалил несколько записей из таблиц bookings.tickets и bookings.bookings

  - прямое соединение таблиц - отбираем номера билетов, по которым есть бронирование
    select ticket_no

    from bookings.tickets t

    inner join bookings.bookings b

    on t.book_ref = b.book_ref limit 10;

  - левостороннее соединение - отбираем номера билетов, по которым нет бронирования

    select ticket_no

    from bookings.tickets t

    left join bookings.bookings b

    on t.book_ref = b.book_ref

    where b.book_ref is null limit 10;

  - кросс соединение двух таблиц - узнаем, из каких аэропортов какие самолеты могли бы вылетать в принципе

    select a.airport_code, b.aircraft_code

    from bookings.airports_data a

    cross join bookings.aircrafts_data b;

  - полное соединение двух таблиц - определяем билеты без бронирования и бронирования без билетов

    select t.ticket_no, b.book_ref

    from bookings.tickets t

    full join bookings.bookings b

    on t.book_ref = b.book_ref

    where t.ticket_no is null or b.book_ref is null;

  - запрос с разными типами соединений - выводим имена аэропортов, по которым отсутствует подтверждения бронирования

    select a.airport_name, count(*)

    from bookings.tickets t

    left join bookings.bookings b

    on t.book_ref = b.book_ref

    inner join bookings.boarding_passes p

    on p.ticket_no = t.ticket_no

    inner join bookings.flights f

    on f.flight_id = p.flight_id

    inner join bookings.airports_data a

    on a.airport_code = f.arrival_airport

    where b.book_ref is null

    group by 1;

