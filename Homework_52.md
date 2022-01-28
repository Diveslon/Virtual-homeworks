# Домашнее задание к занятию "Применение принципов IaaC в работе с виртуальными машинами"

## Задача 1 

- Опишите своими словами основные преимущества применения на практике IaaC паттернов.
- Какой из принципов IaaC является основополагающим?

### Ответ

- Основными преимуществами применения на практике IaaC патернов является полная идентичность любой создаваемой ВМ, поскольку все изменения описываются единым кодом, что приводит к полной идентичности и убирает фактор возможной ошибки, т.е убирает возможность дрейфа конфигурация. Так же такой подход увеличивает скорость прохождения нового продукта от разработчика к потребителю, предоставляет возможность масштабирования и тестирования.
- Основной принцип IaaC- это идемпотентность, т.е. возможность получения идентичного результата при вовторном выполнении.

## Задача 2

- Чем Ansible выгодно отличается от других систем управление конфигурациями?
- Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?

### Ответ
- Ansible выгодно отличается от других систем управление конфигурациями простотой входа и скоростью развёртывания. Нет необходимости устанавливать специальное окружение - он использует текущую SSH инфраструктуру. Так же использует декларативный метод описания и имеет большую раширяемость.
- На мой взгляд более надёжным является push метод работы систем конфигурации, поскольку в этом методе конфигурация целевых серверов отправляется управляющим сервером.


## Задача 3 

Установить на личный компьютер:

- VirtualBox
- Vagrant
- Ansible

### Ответ
```
root@DESKTOP-PF94QOV:~# dpkg-query -l virtualbox vagrant ansible
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name           Version                       Architecture Description
+++-==============-=============================-============-=======================================================================
ii  ansible        2.9.6+dfsg-1                  all          Configuration management, deployment, and task execution system
ii  vagrant        2.2.6+dfsg-2ubuntu3           all          Tool for building and distributing virtualized development environments
ii  virtualbox     6.1.26-dfsg-3~ubuntu1.20.04.2 amd64        x86 virtualization solution - base binaries
root@DESKTOP-PF94QOV:~#
```

## Задача 4
Воспроизвести практическую часть лекции самостоятельно.
Создать виртуальную машину.
Зайти внутрь ВМ, убедиться, что Docker установлен с помощью команды
```
docker pc
```
### Ответ
```
diveslon@diveslon-Ubuntu:~/Netology/Vagrant$ vagrant ssh
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri 28 Jan 2022 05:08:47 AM UTC

  System load:  0.25               Users logged in:          0
  Usage of /:   13.4% of 30.88GB   IPv4 address for docker0: 172.17.0.1
  Memory usage: 24%                IPv4 address for eth0:    10.0.2.15
  Swap usage:   0%                 IPv4 address for eth1:    192.168.192.11
  Processes:    115


This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento
Last login: Fri Jan 28 05:06:32 2022 from 10.0.2.2
vagrant@server1:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
vagrant@server1:~$ 
```

