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
Создаем пустую дирректоию, например postgres и переходим в нее. 

В созданной дирректории создадим файл **docker-compose.yml**, который опишет наш коонтейнер PostgreSQL:

```yml
version: '3.9'

volumes:
  ns2:
  driver: local

services:
  postgres:
    image: postgres:latest # Установим последнюю версию
    container_name: postgres_my # Имя контейнера
    environment: # зададим минимально-необходимые переменные
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
      test: [ "CMD-SHELL", "pg_isready -U postgres_user -d postgres_db" ]
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
docker-compose up -d
```

### Проверка работы контейнера с POstgreSQL
Чтобы проверить, что контейнер установился и запущен, в терминале выполним команду:
```sh
docker ps
```
Запущенный контейре будет выглядеть так:
![shell docker ps](/images/postgres_docker-compose.jpg)


### Подключение к серверу PostgreSQL

При подключении к базам локально достаточно в сервисе подключаться к **localhost:5432**

Подключиться по SSH к серверу, на котором работает PostgreSQL, можно стандартными способами, но важно понимать, что SSH и PostgreSQL — это разные протоколы.

#### 1. Подключение к серверу через SSH
Если у вас есть доступ к серверу через SSH, используйте команду:
```sh
ssh -p number_ssh_port username@server_ip_or_hostname
```

- number_ssh_port - ssh-порт, если он не стандартный 22.
- username — ваше имя пользователя на сервере.
- server_ip_or_hostname — IP-адрес или доменное имя сервера.

#### 2. Подключение к PostgreSQL после входа на сервер
После подключения по SSH вы можете работать с PostgreSQL:

Вариант 1: Через psql (консольный клиент PostgreSQL)
