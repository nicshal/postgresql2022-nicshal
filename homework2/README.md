Описание/Пошаговая инструкция выполнения домашнего задания:

• сделать в GCE инстанс с Ubuntu 20.04
  + поставил

• поставить на нем Docker Engine
  + поставил
    curl -fsSL https://get.docker.com -o get-docker.sh

    sudo sh get-docker.sh

    rm get-docker.sh

    sudo usermod -aG docker $USER

• сделать каталог /var/lib/postgres
  + сделал каталог /var/lib/postgres_test

    sudo su

    mkdir /var/lib/postgres_test

• развернуть контейнер с PostgreSQL 14 смонтировав в него /var/lib/postgres
  + развернул контейнер

    Создал docker-сеть: sudo docker network create pg-net

    sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432  -v /var/lib/postgres_test:/var/lib/postgresql/data postgres:14

    sudo docker ps -a

    fa508c7058d9   postgres:14   "docker-entrypoint.s…"   38 seconds ago   Up 36 seconds   5432/tcp, 0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker

• развернуть контейнер с клиентом postgres
  + развернул контейнер с клиентом

    sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres

• подключится из контейнера с клиентом к контейнеру с сервером и сделать
таблицу с парой строк
   + создал таблицу

     create table docker_test(id int, name text);

     insert into docker_test values(1, 'test1'), (2, 'test2');

• подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов GCP
  + сделал

    psql -p 5432 -U postgres -h 51.250.105.49 -d postgres -W

• удалить контейнер с сервером
  + удалил

    sudo docker stop fa508c7058d9

    sudo docker rm fa508c7058d9

• создать его заново
  + создал

    sudo docker run --name pg-docker --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432  -v /var/lib/postgres_test:/var/lib/postgresql/data postgres:14

    sudo docker ps -a

    62ddf58cdacd   postgres:14   "docker-entrypoint.s…"   7 minutes ago   Up 7 minutes   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   pg-docker

• подключится снова из контейнера с клиентом к контейнеру с сервером
  + подключился

    sudo docker run -it --rm --network pg-net --name pg-client postgres:14 psql -h pg-docker -U postgres

• проверить, что данные остались на месте
  + данные на месте

    postgres=# select * from docker_test;

    id | name

   ----+-------

     1 | test1

     2 | test2

     (2 rows)


• оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами
  + все нормально, проблем не было
