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
