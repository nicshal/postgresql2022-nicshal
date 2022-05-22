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

    create table bookings.bookings_test_2017_05 partition of bookings.bookings_test for values from (timestamptz'2017-05-01 00:00:00') to (timestamptz'2017-05-31 23:59:59' - interval '1 day');

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

