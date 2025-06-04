# Быстрый запуск PostgreSQL через Docker Compose

Быстро и эффективно настроить PostgreSQL с помощью Docker Compose (быстрей чем в Portainere)
Установка PostgreSQL в Portainer из шаблонов хороша, но только несет в себе некотрые условности, которые не позволят быстро установить и подключиьтся к PostgreSQl 
Это связано с тем, что по умолчанию, шаблон установит БД сервер без проброса портов и Вы сможете обращаться к нему только в локальной сети.

Чтобы быстро, без затей и максимально комфорто установить PostgrSQL в контейнер можно воспользоваться или Dokerfale или Docker Compose.
Я выбрал - второе.

## Подготовка
Прежде чем приступить к развертыванию PostgreSQL на вашем локальном компьютере или VPS сервере, необходимо установить Docker и Docker Compose.

Docker — это платформа для разработки, доставки и эксплуатации приложений в контейнерах. 
Контейнеры позволяют упаковывать приложения со всеми необходимыми зависимостями в единую 
стандартизированную единицу, что обеспечивает изоляцию и повторяемость окружения в любой среде.

Docker Compose — инструмент для определения и запуска многоконтейнерных Docker приложений. 
С его помощью вы можете описать все компоненты вашего приложения в файле docker-compose.yml 
и запустить их одной командой. Docker Compose упрощает управление множеством контейнеров, 
позволяя определить зависимости и настройки сети, а также легко настраивать параметры окружения.

Пример настройки docker-compose.yml файла. Этот подход позволяет быстро развернуть PostgreSQL и обеспечит вам надежность в хранении данных используя volumes.

### **Шаг 1**: Создание Docker Compose файла
Создаем пустую директорию, например postgres и переходим в нее. 

В созданной директории создадим файл **docker-compose.yml**, который опишет наш контейнер PostgreSQL:

```yml
volumes:
  ns2:
    driver: local

services:
  postgres:
    image: postgres:latest
    container_name: postgres_my
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: You-PoStgrSuser_password
      POSTGRES_DB: usr1_db
      PGDATA: /var/lib/postgresql/data/pgdata
    ports:
      - "5432:5432"
    volumes:
      - ns2:/var/lib/postgresql/data
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    command: >
      postgres -c max_connections=1000
               -c shared_buffers=256MB
               -c effective_cache_size=768MB
               -c maintenance_work_mem=64MB
               -c checkpoint_completion_target=0.7
               -c wal_buffers=16MB
               -c default_statistics_target=100
    healthcheck:
      test: [ "CMD-SHELL", "pg_isready -U postgres -d usr1_db" ]
      interval: 30s
      timeout: 10s
      retries: 5
    restart: unless-stopped
    tty: true
    stdin_open: true
```

#### Краткий обзор Docker Compose файла
1. services/postgres:
  - image: используемая Docker-образ PostgreSQL, в данном случае postgres:latest.
  - container_name: имя контейнера, в котором будет запущен PostgreSQL.
  - environment: переменные окружения для настройки PostgreSQL (пользователь, пароль, имя базы данных - не забудьте указать свои).
  - ports: проброс портов, где "5430:5432" означает, что порт PostgreSQL внутри контейнера (5432) проброшен на порт хоста (5430). Это значит что для подключения к постгрес нужно будет прописывать порт 5430.
  - volumes: монтируем локальный каталог ./pgdata внутрь контейнера для сохранения данных PostgreSQL.
  - deploy: определяет ресурсы и стратегию развертывания для Docker Swarm (необязательно для стандартного использования Docker Compose).
  - command: дополнительные параметры командной строки PostgreSQL для настройки параметров производительности.
  - healthcheck: проверка состояния PostgreSQL с использованием pg_isready.
  - restart, tty, stdin_open: настройки перезапуска контейнера и взаимодействия с ним через терминал.

2. volumes/pgdata:
  - Определяет том pgdata, который используется для постоянного хранения данных PostgreSQL.

### Запуск PostgreSQL
Чтобы развернуть PostgreSQL с помо0щью этого файла Docker Compose, выполним команду в каталоге с файлом docker-compose.yml:
```sh
docker compose up -d
```

### Проверка работы контейнера с POstgreSQL
Чтобы проверить, что контейнер установился и запущен, в терминале выполним команду:
```sh
docker ps
```
Запущенный контейнер будет выглядеть так:
![shell docker ps](/images/postgres_docker-compose.jpg)


### Подключение к серверу PostgreSQL

При подключении к базам локально достаточно в сервисе подключаться к **localhost:5432**

Если PostgreSQL запущен в Docker-контейнере, есть несколько способов подключиться к нему:

#### 1.Подключение через psql внутри контейнера
Если вам нужно просто выполнить SQL-запросы, можно зайти прямо в контейнер и использовать psql:
```sh
docker exec -it <container_name_or_id> psql -U <postgres_user> -d <database_name>
```

Если нужно войти в контейнер и потом запустить psql:
```bash
docker exec -it postgres_container bash
psql -U postgres
```

Чтобы вывести список баз подключившись к контейнеру и фодя в psql
```sh
\l
```
Вывод:
```sh
                                  List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges   
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 usr1_db   | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
(4 rows)
```

или через SQL-запрос:

```sql
SELECT datname FROM pg_database;
```
Вывод:
```sh
  datname  
-----------
 postgres
 usr1_db
 template1
 template0
(4 rows)
```
Дополнительные фильтры (если нужно исключить системные БД)
```sh
SELECT datname FROM pg_database 
WHERE datname NOT IN ('template0', 'template1', 'postgres');
```

#### Проверка конкретной базы

Чтобы проверить существование базы данных `pgmy_db` и её доступность из `psql`, можно выполнить следующие запросы:

##### 1.  Проверить конкретно базу `pgmy_db`:
```sql
SELECT datname FROM pg_database WHERE datname = 'pgmy_db';
```
Если база существует, запрос вернет одну строку с её именем.

##### 2. Проверить подключение к конкретной базе (если вы уже подключены к другой БД):
```sql
\c pgmy_db
```
Если подключение успешно, вы увидите сообщение типа:
```
You are now connected to database "pgmy_db" as user "your_user".
```

##### 3. Проверить доступность и выполнить простой тестовый запрос (если подключились к pgmy_db):
```sql
SELECT 1;
```
Должен вернуть:
```
 ?column? 
----------
        1
(1 row)
```

##### Полный пример проверки из командной строки:
```bash
psql -U postgres -c "SELECT datname FROM pg_database WHERE datname = 'pgmy_db'"
```

Если база существует и доступна, вы увидите:
```
 datname  
----------
 pgmy_db
(1 row)
```

Если базы нет или нет доступа, результат будет пустым или вы получите сообщение об ошибке доступа.

#### 2. Подключение с локальной машины (если порт проброшен)
При запуске контейнера PostgreSQL обычно пробрасывают порт (5432 по умолчанию):
Например если постгрес был запущен docker-ом:
```sh
docker run --name postgres_container -e POSTGRES_PASSWORD=mysecretpassword -p 5432:5432 -d postgres
```
или как описано выше, то порты уже проброшены и подключиться к ним локально можно следующим образом:
```sh
psql -h localhost -p 5432 -U postgres -d postgres
```
Параметры:
* -h localhost — хост (если Docker на локальной машине).
* -p 5432 — порт (который проброшен в docker run).
* -U postgres — пользователь PostgreSQL.
* -d postgres — база данных (по умолчанию postgres).

#### 3. Подключение из другой сети Docker (если контейнеры в одной сети)
Если PostgreSQL в одной Docker-сети с другим контейнером, можно подключиться по имени сервиса:

```bash
psql -h postgres_container -p 5432 -U postgres -d postgres
```
(Имя postgres_container должно быть указано в docker-compose.yml или при запуске docker run --name.)

#### 4. Использование DBeaver, pgAdmin или другого GUI-клиента
Если порт проброшен (-p 5432:5432), можно подключиться из графического клиента:
* Хост: localhost
* Порт: 5432
* Пользователь: postgres (или другой, если создан)
* Пароль: Указан в POSTGRES_PASSWORD

### Возможные проблемы.

1. Ошибка: "Connection refused"

* Проверьте, что порт проброшен (-p 5432:5432).
* Убедитесь, что контейнер запущен:

```bash
docker ps
```

2. Ошибка аутентификации

* Проверьте POSTGRES_USER и POSTGRES_PASSWORD.
* Если пароль не задан, попробуйте postgres (по умолчанию).

3. PostgreSQL не слушает внешние подключения
   
Если в логах видно listen_addresses = 'localhost', нужно изменить postgresql.conf (внутри контейнера):

```bash
docker exec -it postgres_container bash
echo "listen_addresses = '*'" >> /var/lib/postgresql/data/postgresql.conf
exit
docker restart postgres_container
```

### Если что-то не работает — проверьте логи PostgreSQL:

```bash
docker logs postgres_container
```

### Если что-то не работает — проверьте логи PostgreSQL:

```bash
docker logs postgres_container
```
