# Prometheus + Grafana
Устанавливать связку Prometheus + Grafana

### Запуск контейнера **node-exporter**

Установите **node_exporter** на все хосты, которые вы хотите отслеживать. 

Это руководство покажет вам, как установить его локально.

```sh
docker run -d \
 --net="host" \
 --pid="host" \
 -v "/:/host:ro,rslave" \
 quay.io/prometheus/node-exporter:latest \
 --path.rootfs=/host
 ```

При локальном запуске node_exporter перейдите по адресу , [http://localhost:9100/metrics](http://localhost:9100/metrics) чтобы проверить, экспортирует ли он метрики.

## Вариант 1. Запуск Prometheus + Grafana посредством Docker

### Запуск контейнера Prometheus

Предварительно настроим конфигурационный файл **Prometheus.yml**

Измените файл конфигурации **Prometheus**, чтобы отслеживать хосты, на которых установлен **node_exporter**

```sh
nano /prometheus/prometheus.yml
```

```yaml
global:
 scrape_interval: 5s
 external_labels:
   monitor: 'node'

scrape_configs:
 - job_name: 'prometheus'  
   static_configs:  
     - targets: ['IP-address:9090'] ## IP Address of the localhost
 - job_name: 'node-exporter'  
   static_configs:  
     - targets: ['1IP-address:9100'] ## IP Address of the localhost node-exporter
```

запуск контейнера **prometheus**

```sh
docker run \
 -d --name prometheus \
 -p 9090:9090 \
 -v $PWD/prometheus.yml:/etc/prometheus/prometheus.yml \
 -v prometheus-data:/prometheus \
 prom/prometheus:latest
```

### Остановка контейнера Prometheus
Чтобы остановить контейнер Prometheus, выполните следующую команду:

```sh
# The `docker ps` command shows the processes running in Docker
docker ps

# This will display a list of containers that looks like the following:

CONTAINER ID   IMAGE                   COMMAND                  CREATED      STATUS       PORTS    NAMES
78858c384pc6   prom/prometheus:latest  "/bin/prometheus --c…"   2 days ago   Up 2 hours  0.0.0.0:9090->9090/tcp, prometheus

# To stop the prometheus container run the command
# docker stop CONTAINER-ID or use
# docker stop NAME, which is `prometheus` as previously defined
docker stop prometheus
```

### Запуск контейнера Grafana

```sh
docker run -d --name=grafana -p 3000:3000 grafana/grafana-enterprise
```

**(Рекомендуется!)** Или используйте тома Docker, если вы хотите, чтобы Docker Engine управлял томом хранилища.
```sh
# start grafana
docker run -d -p 3000:3000 --name=grafana \
  --volume grafana-storage:/var/lib/grafana \
  grafana/grafana-enterprise
```

### Остановка контейнера Grafana
Чтобы остановить контейнер Grafana, выполните следующую команду:

```sh
# The `docker ps` command shows the processes running in Docker
docker ps

# This will display a list of containers that looks like the following:
CONTAINER ID   IMAGE  COMMAND   CREATED  STATUS   PORTS    NAMES
cd48d3994968   grafana/grafana-enterprise   "/run.sh"   8 seconds ago   Up 7 seconds   0.0.0.0:3000->3000/tcp   grafana

# To stop the grafana container run the command
# docker stop CONTAINER-ID or use
# docker stop NAME, which is `grafana` as previously defined
docker stop grafana
```

## Вариант 2. Запуск Prometheus + Grafana посредством Docker-Compose

Установим Docker-compose

```sh
apt  install docker-compose
```

### Установка Prometheus + Node Exporter

Установим Prometheus и для сбора статистики с хост системы(системы на которой установлен сервер Grafana) дополнительно установим Node Exporter.

Для упрощения установки мы создадим в дирректории ``/prometeus`` в которой будет располагаться файл конфигурации **prometheus.yml** файл **docker-compose.yml** следующим содержимым:

```yml
version: '3.3'

networks:
  monitoring:
    driver: bridge
    
volumes:
  prometheus_data: {}

services:
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    restart: unless-stopped
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.rootfs=/rootfs'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    ports:
      - 9100:9100
    networks:
      - monitoring
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    restart: unless-stopped
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--web.enable-lifecycle'
    ports:
      - 9090:9090
    networks:
      - monitoring
```

создадим файл prometheus.yml со следующим содержимым:

```sh
global:
 scrape_interval: 5s
 external_labels:
   monitor: 'node'

scrape_configs:
 - job_name: 'prometheus'  
   static_configs:  
     - targets: ['IP-address:9090'] ## IP Address of the localhost
 - job_name: 'node-exporter'  
   static_configs:  
     - targets: ['1IP-address:9100'] ## IP Address of the localhost node-exporter
```

выполним установку

```sh
docker-compose up -d
```

Если в  **prometheus.yml** добавили еще узел. нужно пересобрать контейнер:

ПЕРЕЗАПУСК контейнеров в docker-compose

```sh
docker-compose down && sudo docker-compose up -d
```