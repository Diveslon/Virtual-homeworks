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

USER root

RUN \
    yum install -y wget && \
    yum install -y perl-Digest-SHA && \
    yum install -y java-11-openjdk && \
    yum install -y java-11-openjdk-devel 
RUN useradd elasticsearch -c "elastisearch user" -d /home/elasticsearch
RUN \
    mkdir -p /local/elasticsearch-8.1.1 && \
    chown elasticsearch:elasticsearch /var/lib &&\
    chown elasticsearch:elasticsearch /local/elasticsearch-8.1.1
USER elasticsearch
ENV ES_HOME=/local/elasticsearch-8.1.1
ENV ES_CONFIG=/local/elasticsearch-8.1.1/config/elasticsearch.yml

EXPOSE 9200
EXPOSE 9300
    
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
    echo "path.data: /var/lib" >> $ES_CONFIG

CMD $ES_HOME/bin/elasticsearch
```
- ссылка на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде
```
[elastic@35677879033b /]$ curl -ku elastic https://localhost:9200
Enter host password for user 'elastic':
{
  "name" : "netology_test",
  "cluster_name" : "netology_cluster",
  "cluster_uuid" : "5FY8px9vRyScVjbSDvFNZA",
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

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
