
# Домашнее задание к занятию "5.3. Введение. Экосистема. Архитектура. Жизненный цикл Docker контейнера"

## Задача 1

Сценарий выполения задачи:

- создайте свой репозиторий на https://hub.docker.com;
- выберете любой образ, который содержит веб-сервер Nginx;
- создайте свой fork образа;
- реализуйте функциональность:
запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на https://hub.docker.com/username_repo.

### Ответ: 
https://hub.docker.com/repository/docker/diveslon/hometask53

## Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос:
"Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"

Детально опишите и обоснуйте свой выбор.

--

Сценарий:

- Высоконагруженное монолитное java веб-приложение;
- Nodejs веб-приложение;
- Мобильное приложение c версиями для Android и iOS;
- Шина данных на базе Apache Kafka;
- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
- Мониторинг-стек на базе Prometheus и Grafana;
- MongoDB, как основное хранилище данных для java-приложения;
- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.

## Задача 3

- Запустите первый контейнер из образа ***centos*** c любым тэгом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```;
- Добавьте еще один файл в папку ```/data``` на хостовой машине;
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.

### Ответ
```
root@diveslon-Ubuntu:~# 
root@diveslon-Ubuntu:~# docker run --name test-centos -v /root/data/:/root/share/data -d centos tail -f /dev/null
16f7ad137fdfc11fc8a24f8365b5da10fbb8853a9c0d7fb4ccb8aa4ab74771da
root@diveslon-Ubuntu:~# docker run --name test-debian -v /root/data/:/root/share/data -d debian tail -f /dev/null
f08bae465d3b5f6bf4db12fb050ecbbae564e08a8a8fd09092807296229e0089
root@diveslon-Ubuntu:~# docker ps
CONTAINER ID   IMAGE     COMMAND               CREATED          STATUS          PORTS     NAMES
f08bae465d3b   debian    "tail -f /dev/null"   10 seconds ago   Up 9 seconds              test-debian
16f7ad137fdf   centos    "tail -f /dev/null"   33 seconds ago   Up 32 seconds             test-centos
root@diveslon-Ubuntu:~# docker exec -it test-centos bash
[root@16f7ad137fdf /]# cd /root/share/data/
[root@16f7ad137fdf data]# echo 'This is a first test!' >> test1.txt
[root@16f7ad137fdf data]# exit
root@diveslon-Ubuntu:~# cd /root/data/
root@diveslon-Ubuntu:~/data# echo 'This is a second test!' >> test2.txt
root@diveslon-Ubuntu:~/data# docker exec -it test-debian bash
root@f08bae465d3b:/# cd /root/share/data/
root@f08bae465d3b:~/share/data# ls
test1.txt  test2.txt
root@f08bae465d3b:~/share/data# 
exit
root@diveslon-Ubuntu:~/data#
```

## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

Соберите Docker образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.

### Ответ
https://hub.docker.com/r/diveslon/ansible


---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
