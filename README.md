<img width="1144" height="576" alt="image" src="https://github.com/user-attachments/assets/9629db52-c3ba-4470-9c4f-ba15b91bcbf8" /># Домашнее задание к занятию «ELK» - Солдатов Дмитрий

## Задание 1. Elasticsearch

Установите и запустите Elasticsearch, после чего поменяйте параметр cluster_name на случайный.

**Решение:**
Для развертывания использовался Docker. Параметр `cluster.name` был изменен в конфигурации сервиса Elasticsearch (`docker-compose.yml`):

~~~yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - cluster.name=my-super-random-cluster-999 
~~~

Скриншот выполнения команды `curl -X GET 'localhost:9200/_cluster/health?pretty'`, где виден нестандартный `cluster_name`:

<img width="939" height="421" alt="image" src="https://github.com/user-attachments/assets/8a65c0b7-e756-4220-ab7c-79c5b73d2f16" />

---

## Задание 2. Kibana

Установите и запустите Kibana.

**Решение:**
Kibana была запущена в Docker. В интерфейсе Dev Tools выполнен запрос к кластеру.

Скриншот интерфейса Kibana на странице `http://<ip сервера>:5601/app/dev_tools#/console` с выполненным запросом `GET /_cluster/health?pretty`:

<img width="1868" height="930" alt="image" src="https://github.com/user-attachments/assets/ebf493db-d9b7-401e-98f8-f1aeb13ae273" />


---

## Задание 3. Logstash

Установите и запустите Logstash и Nginx. С помощью Logstash отправьте access-лог Nginx в Elasticsearch.

**Решение:**
Создан конфигурационный файл `logstash.conf` для чтения логов Nginx и отправки их в Elasticsearch:

~~~ruby
input {
  file {
    path => "/var/log/nginx/access.log"
    start_position => "beginning"
  }
}
output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "nginx-logstash-%{+YYYY.MM.dd}"
  }
}
~~~

Скриншот интерфейса Kibana, на котором видны логи Nginx, собранные через Logstash:

<img width="1874" height="851" alt="image" src="https://github.com/user-attachments/assets/bb273b8f-ec06-4630-b11c-1aca2f155dc2" />


---

## Задание 4. Filebeat

Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat.

**Решение:**
Был запущен Filebeat, который читает логи Nginx напрямую и отправляет их в Elasticsearch (минуя Logstash). Конфигурация `filebeat.yml`:

~~~yaml
filebeat.inputs:
- type: filestream
  id: nginx-logs
  paths:
    - /var/log/nginx/access.log

output.elasticsearch:
  hosts: ["http://elasticsearch:9200"]
  index: "nginx-filebeat-%{+yyyy.MM.dd}"
~~~

Скриншот интерфейса Kibana, на котором видны логи Nginx, отправленные через Filebeat:

<img width="1864" height="840" alt="image" src="https://github.com/user-attachments/assets/5509549e-5081-4fee-a46c-67bb1f22808e" />

