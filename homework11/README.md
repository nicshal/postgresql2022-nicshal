Описание/Пошаговая инструкция выполнения домашнего задания:
Секционировать большую таблицу из демо базы flights

  - скопировал на сервер архив с demo-базой (https://postgrespro.com/education/demo-medium-en.zip)

  - распаковал архив:

    unzip demo-medium-en.zip

  - создал базу demo:

    create database demo;

  - перешел в базу demo и залил скрипт:

    demo=# \i /home/nicshal/demo-medium-en-20170815.sql

  - создал копию таблицы bookings.bookings, добавив секционирование

    create table bookings.bookings_test
    (
        book_ref     char(6)                  not null,
        book_date    timestamp with time zone not null,
        total_amount numeric(10, 2)           not null
    ) partition by range (book_date);

  - создал несколько секций:

    create table bookings.bookings_test_2017_05 partition of bookings.bookings_test for values from (timestamptz'2017-05-01 00:00:00') to (timestamptz'2017-06-01 23:59:59' - interval '1 day');

    create table bookings.bookings_test_2017_06 partition of bookings.bookings_test for values from (timestamptz'2017-06-01 00:00:00') to (timestamptz'2017-07-01 23:59:59' - interval '1 day');

    create table bookings.bookings_test_2017_07 partition of bookings.bookings_test for values from (timestamptz'2017-07-01 00:00:00') to (timestamptz'2017-08-01 23:59:59' - interval '1 day');

    create table bookings.bookings_test_2017_08 partition of bookings.bookings_test for values from (timestamptz'2017-08-01 00:00:00') to (timestamptz'2017-09-01 23:59:59' - interval '1 day');

    create table bookings.bookings_test_default partition of bookings.bookings_test default;

  - кокируем данные:

    insert into bookings.bookings_test(book_ref, book_date, total_amount) select book_ref, book_date, total_amount from bookings.bookings;

  - проверяем, что данные появились в секции:

    select * from bookings.bookings_test_2017_06 limit 5;

      0000C9,2017-06-30 12:52:00.000000 +00:00,54600.00

      000112,2017-06-14 20:50:00.000000 +00:00,22000.00

      00015D,2017-06-05 17:26:00.000000 +00:00,281500.00

      0001CE,2017-06-25 12:02:00.000000 +00:00,267400.00

      0002CD,2017-06-23 02:58:00.000000 +00:00,32800.00

  - делаем выборку по диапазону дат

    explain analyse

    select * from bookings.bookings_test

    where book_date between '2017-06-09' and '2017-06-10';

  - в плане видим, что используется секция:

      Seq Scan on bookings_test_2017_06 bookings_test  (cost=0.00..3531.19 rows=5296 width=21) (actual time=0.007..9.529 rows=5497 loops=1)

      Filter: ((book_date >= '2017-06-09 00:00:00+00'::timestamp with time zone) AND (book_date <= '2017-06-10 00:00:00+00'::timestamp with time zone))

      Rows Removed by Filter: 159716

      Planning Time: 0.159 ms

      Execution Time: 9.714 ms

  - сделаем индекс по дате:

    create index bookings_book_date_idx on bookings.bookings (book_date);

  - делаем выборку по диапазону дат

    explain analyse

    select * from bookings.bookings_test

    where book_date between '2017-06-09' and '2017-06-10';

  - в плане видим использование индекса:

      Bitmap Heap Scan on bookings_test_2017_06 bookings_test  (cost=90.58..1223.02 rows=5296 width=21) (actual time=0.464..2.396 rows=5497 loops=1)

      Recheck Cond: ((book_date >= '2017-06-09 00:00:00+00'::timestamp with time zone) AND (book_date <= '2017-06-10 00:00:00+00'::timestamp with time zone))

      Heap Blocks: exact=1048

      ->  Bitmap Index Scan on bookings_test_2017_06_book_date_idx  (cost=0.00..89.26 rows=5296 width=0) (actual time=0.326..0.326 rows=5497 loops=1)

          Index Cond: ((book_date >= '2017-06-09 00:00:00+00'::timestamp with time zone) AND (book_date <= '2017-06-10 00:00:00+00'::timestamp with time zone))

      Planning Time: 0.264 ms

      Execution Time: 2.579 ms

  - решили добавить еще одну секцию:

    alter table bookings.bookings_test detach partition bookings.bookings_test_default;

    create table bookings.bookings_test_2017_04 partition of bookings.bookings_test for values from (timestamptz'2017-04-01 00:00:00') to (timestamptz'2017-05-01 23:59:59' - interval '1 day');

    create table bookings.bookings_test_default2 partition of bookings.bookings_test default;

    insert into bookings.bookings_test select * from bookings.bookings_test_default;

    drop table bookings.bookings_test_default;


