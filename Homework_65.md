# Домашнее задание к занятию "6.5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib`
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- возможно вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения

Далее мы будем работать с данным экземпляром elasticsearch.

### Ответ
- текст Dockerfile манифеста
```
FROM centos:7

MAINTAINER Alexander Lon <5227676@gmail.com>

EXPOSE 9200

USER root

RUN \
    yum install -y wget && \
    yum install -y perl-Digest-SHA && \
    yum install -y java-11-openjdk  && \
    yum install -y java-11-openjdk-devel
RUN useradd elastic -c "elastisearch user" -d /home/elastic
RUN \
    mkdir -p /local/elasticsearch && \
    mkdir -p /var/lib/elasticsearch && \
    chown elastic /var/lib/elasticsearch &&\
    chown elastic /local/elasticsearch
USER elastic

ENV ES_HOME=/local/elasticsearch
ENV ES_CONFIG=/local/elasticsearch/config/elasticsearch.yml

LABEL io.k8s.description="Elasticsearch container" \
      io.openshift.expose-services="9200:http, 9300:http" \
      io.openshift.tags="elasticsearch" \
      architecture=x86_64 \
      name="openshift3/elasticsearch"
    
RUN \
    cd /tmp && \
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.1.1-linux-x86_64.tar.gz && \
    wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.1.1-linux-x86_64.tar.gz.sha512 && \
    shasum -a 512 -c elasticsearch-8.1.1-linux-x86_64.tar.gz.sha512 && \
    tar -xzf elasticsearch-8.1.1-linux-x86_64.tar.gz && \
    rm elasticsearch-8.1.1-linux-x86_64.tar.gz && \
    rm elasticsearch-8.1.1-linux-x86_64.tar.gz.sha512 && \
    mv /tmp/elasticsearch-8.1.1/* $ES_HOME

RUN \
    echo "cluster.name: netology_cluster" >> $ES_CONFIG && \
    echo "node.name: netology_test" >> $ES_CONFIG && \
    echo "path.data: /var/lib/elasticsearch" >> $ES_CONFIG

CMD $ES_HOME/bin/elasticsearch
```
- [ссылка на образ в репозитории dockerhub ](https://hub.docker.com/repository/docker/diveslon/homework65/tags)
- ответ `elasticsearch` на запрос пути `/` в json виде
```
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic https://localhost:9200
Enter host password for user 'elastic':
{
  "name" : "netology_test",
  "cluster_name" : "netology_cluster",
  "cluster_uuid" : "Ya6Ngs6qQy2fSXzt3Z96dQ",
  "version" : {
    "number" : "8.1.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "d0925dd6f22e07b935750420a3155db6e5c58381",
    "build_date" : "2022-03-17T22:01:32.658689558Z",
    "build_snapshot" : false,
    "lucene_version" : "9.0.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

### Ответ
Ознакомтесь с документацией]и добавьте в `elasticsearch` 3 индекса
```
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic -X PUT "https://localhost:9200/ind-1" -H 'Content-Type: application/json' -d'
> {
>          "settings": {
>            "index": {
>              "number_of_shards": 1,
>              "number_of_replicas": 0
>            }
>           }
>         }
>       '
Enter host password for user 'elastic':
{"acknowledged":true,"shards_acknowledged":true,"index":"ind-1"}

diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic -X PUT "https://localhost:9200/ind-2" -H 'Content-Type: application/json' -d'
{
         "settings": {
           "index": {
             "number_of_shards": 2,
             "number_of_replicas": 1
           }
          }
        }
      '
Enter host password for user 'elastic':
{"acknowledged":true,"shards_acknowledged":true,"index":"ind-2"}

diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic -X PUT "https://localhost:9200/ind-3" -H 'Content-Type: application/json' -d'
> {
>          "settings": {
>            "index": {
>              "number_of_shards": 4,
>              "number_of_replicas": 2
>            }
>           }
>         }
>       '
Enter host password for user 'elastic':
{"acknowledged":true,"shards_acknowledged":true,"index":"ind-3"}
```
Получите список индексов и их статусов, используя API
```
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic  https://localhost:9200/_cat/indices
Enter host password for user 'elastic':
green  open ind-1 pJJ2M0zCS1Os1-evh2MkrQ 1 0 0 0 225b 225b
yellow open ind-3 i_ag2-KBQ9e5s5yapxEAvw 4 2 0 0 900b 900b
yellow open ind-2 rGxeMCU2R8-pmF1OLZP-EQ 2 1 0 0 450b 450b
```
Получите состояние кластера `elasticsearch`, используя API.
```
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic  https://localhost:9200/_cluster/health?pretty
Enter host password for user 'elastic':
{
  "cluster_name" : "netology_cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 9,
  "active_shards" : 9,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 47.368421052631575
}
```
- Часть индексов жёлтые, поскольку созданы с количеством реплик и шард больше, чем фактически есть
- Сам кластер жёлтый, так как unassigned_shards > 0

Удалите все индексы
```
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic  -X DELETE https://localhost:9200/ind-1
Enter host password for user 'elastic':
{"acknowledged":true}
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic  -X DELETE https://localhost:9200/ind-2
Enter host password for user 'elastic':
{"acknowledged":true}
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic  -X DELETE https://localhost:9200/ind-3
Enter host password for user 'elastic':
{"acknowledged":true}
```

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

### Ответ
Доработаем Dockerfile, добавив создание директории и назначение прав ( пока ещё под рутом)
```
RUN \
    mkdir -p /local/elasticsearch && \
    mkdir -p /var/lib/elasticsearch && \
    mkdir -p /local/elasticsearch/snapshots && \
    chown elastic /var/lib/elasticsearch && \
    chown elastic /local/elasticsearch/snapshots && \
    chown elastic /local/elasticsearch
```
и под пользователем elastic
```
RUN \
    echo "cluster.name: netology_cluster" >> $ES_CONFIG && \
    echo "node.name: netology_test" >> $ES_CONFIG && \
    echo "path.repo: /local/elasticsearch/snapshots" >> $ES_CONFIG && \
    echo "path.data: /var/lib/elasticsearch" >> $ES_CONFIG
```
Используя API зарегистрируйте данную директорию как `snapshot repository` c именем `netology_backup`
```
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic -X PUT "https://localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d' 
{ 
"type": "fs", 
"settings": {
 "location": "/local/elasticsearch/snapshots" 
}
 } '
Enter host password for user 'elastic':
{
  "acknowledged" : true
}

```
Создайте индекс `test` с 0 реплик и 1 шардом и приведите в ответе список индексов
```
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic  https://localhost:9200/_cat/indices
Enter host password for user 'elastic':
green open test z5_H6SEQR1acvY89_q_EMw 1 0 0 0 225b 225b
```
Создайте `snapshot` состояния кластера `elasticsearch` и приведите в ответе список файлов в директории со `snapshot`ами.
```
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic -X PUT https://localhost:9200/_snapshot/netology_backup/my_snapshot?wait_for_completion=true
Enter host password for user 'elastic':
{"snapshot":{"snapshot":"my_snapshot","uuid":"agIcXu2RRg287Yoz5zfF4g","repository":"netology_backup","version_id":8010199,"version":"8.1.1","indices":["test",".security-7",".geoip_databases"],"data_streams":[],"include_global_state":true,"state":"SUCCESS","start_time":"2022-04-02T16:32:08.181Z","start_time_in_millis":1648917128181,"end_time":"2022-04-02T16:32:09.181Z","end_time_in_millis":1648917129181,"duration_in_millis":1000,"failures":[],"shards":{"total":3,"failed":0,"successful":3},"feature_states":[{"feature_name":"geoip","indices":[".geoip_databases"]},{"feature_name":"security","indices":[".security-7"]}]}}
diveslon@diveslon-Ubuntu:~/homework65$ sudo docker exec -it es2 ls -la /local/elasticsearch/snapshots/
total 48
drwxr-xr-x 1 elastic root     4096 Apr  2 16:32 .
drwxr-xr-x 1 elastic root     4096 Apr  2 16:09 ..
-rw-r--r-- 1 elastic elastic  1096 Apr  2 16:32 index-0
-rw-r--r-- 1 elastic elastic     8 Apr  2 16:32 index.latest
drwxr-xr-x 5 elastic elastic  4096 Apr  2 16:32 indices
-rw-r--r-- 1 elastic elastic 18424 Apr  2 16:32 meta-agIcXu2RRg287Yoz5zfF4g.dat
-rw-r--r-- 1 elastic elastic   386 Apr  2 16:32 snap-agIcXu2RRg287Yoz5zfF4g.dat

```
Удалите индекс `test` и создайте индекс `test-2`. Приведите в ответе список индексов.
```
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic  https://localhost:9200/_cat/indices
Enter host password for user 'elastic':
green open test-2 F56xk21cQdKnflhQg6THzA 1 0 0 0 225b 225b
```
Восстановите состояние кластера `elasticsearch` из `snapshot`, созданного ранее. Приведите в ответе запрос к API восстановления и итоговый список индексов.
```
diveslon@diveslon-Ubuntu:~/homework65$ curl -ku elastic -X POST 'https://localhost:9200/_snapshot/netology_backup/my_snapshot/_restore?wait_for_completion=true'
Enter host password for user 'elastic':
{"snapshot":{"snapshot":"my_snapshot","indices":["test"],"shards":{"total":1,"failed":0,"successful":1}}}

diveslon-Ubuntu:~/homework65$ curl -ku elastic  https://localhost:9200/_cat/indices
Enter host password for user 'elastic':
green open test-2 F56xk21cQdKnflhQg6THzA 1 0 0 0 225b 225b
green open test   uJLSE4jbRuKNIW0CFV0BiQ 1 0 0 0 225b 225b

```
