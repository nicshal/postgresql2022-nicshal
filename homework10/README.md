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







  2.Прислать текстом результат команды explain, в которой используется данный индекс

  3.Реализовать индекс для полнотекстового поиска

  4.Реализовать индекс на часть таблицы или индекс на поле с функцией

  5.Создать индекс на несколько полей

  6.Написать комментарии к каждому из индексов

  7.Описать что и как делали и с какими проблемами столкнулись

2 вариант: В результате выполнения ДЗ вы научитесь пользоваться различными вариантами соединения таблиц.

  1.Реализовать прямое соединение двух или более таблиц

  2.Реализовать левостороннее (или правостороннее) соединение двух или более таблиц

  3.Реализовать кросс соединение двух или более таблиц

  4.Реализовать полное соединение двух или более таблиц

  5.Реализовать запрос, в котором будут использованы разные типы соединений

  6.Сделать комментарии на каждый запрос

  7.К работе приложить структуру таблиц, для которых выполнялись соединения