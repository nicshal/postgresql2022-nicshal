Описание/Пошаговая инструкция выполнения домашнего задания:

создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE типа e2-medium в default VPC в любом регионе и зоне, например us-central1-a
 + сделал

поставьте на нее PostgreSQL 14 через sudo apt
 + сделал

проверьте что кластер запущен через sudo -u postgres pg_lsclusters
 + кластер запущен

зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым postgres=# create table test(c1 text); postgres=# insert into test values('1'); \q
 + таблицу сделал

остановите postgres например через sudo -u postgres pg_ctlcluster 14 main stop
 + остановил

создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB
 + создал диск размером 5 Гб (Compute Cloud -> Диски -> Создать диск)

добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
 + добавил - /dev/vdb

     NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT

     vda    252:0    0  15G  0 disk

     ├─vda1 252:1    0   1M  0 part

     └─vda2 252:2    0  15G  0 part /

      vdb    252:16   0   5G  0 disk


проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
 + выполнено

     Filesystem      Size  Used Avail Use% Mounted on

     /dev/vda2        15G  3.1G   12G  22% /

     /dev/vdb1       4.9G   20M  4.6G   1% /mnt/data


сделайте пользователя postgres владельцем /mnt/data - sudo chown -R postgres:postgres /mnt/data/

перенесите содержимое /var/lib/postgres/14 в /mnt/data - sudo mv /var/lib/postgresql/14 /mnt/data
 + сделал

попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
 + попытался запустить

напишите получилось или нет и почему
 + Error: /var/lib/postgresql/14/main is not accessible or does not exist

задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/10/main который надо поменять и поменяйте его
 + параметр data_directory в postgresql.conf

напишите что и почему поменяли
 + data_directory = '/mnt/data/14/main'

попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 14 main start
 + попытался

напишите получилось или нет и почему
 + кластер запущен

     14  main    5432 online postgres /mnt/data/14/main /var/log/postgresql/postgresql-14-main.log

зайдите через через psql и проверьте содержимое ранее созданной таблицы
 + данные обнаружены

задание со звездочкой *: не удаляя существующий GCE инстанс сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.
 + на первой машине выполнил sudo umount /mnt/data/
 + отсоединил диск от первой машины
 + подключил диск ко второй машине
 + выполнил на второй машине:
   - sudo mkdir -p /mnt/data
   - sudo mount -o defaults /dev/vdb1 /mnt/data
   - sudo chown -R postgres:postgres /mnt/data/
   - отредактировал в postgresql.conf: data_directory = '/mnt/data/14/main'
   - стартовал кластер sudo -u postgres pg_ctlcluster 14 main start
   - зашел в psql: sudo -u postgres psql
   - увидел данные в таблице test

