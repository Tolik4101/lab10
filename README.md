Отчет по лабораторной работе "Изучение Ansible"

## Часть 1 (подготовительная):

В ходе данной части лабораторной работы я выполнил следующие действия:

1. Развернул две виртуальные машины с операционной системой Linux. Первая машина (ВМ-1) стала управляющей, а вторая машина (ВМ-2) – управляемой.

2. На ВМ-1 установил Ansible, следуя официальной документации по установке для моей операционной системы.

3. Сгенерировал пару SSH-ключей на ВМ-1 с помощью утилиты ```ssh-keygen```. Для этого выполнены следующие команды в терминале ВМ-1:
```sh
$ ssh-keygen -t rsa
```
При этом были указаны необходимые флаги для генерации ключей, такие как директория сохранения ключей и выбор типа ключа (в данном случае RSA).
4. Передал открытый SSH-ключ с ВМ-1 на ВМ-2 для аутентификации. Для этого использовал утилиту ssh-copy-id, которая установлена на ВМ-1. Выполнил следующую команду в терминале ВМ-1:
```sh
$ ssh-copy-id <username>@<VM-2_IP>
```

После ввода пароля от пользователя на ВМ-2, открытый ключ был скопирован на ВМ-2.

## Часть 2 (основная):

В данной части работы я создал следующие файлы для Ansible:
1. Файл инвенторизации (inventory), в котором указал информацию о хостах (ВМ-1 и ВМ-2). Пример содержимого файла:
```php
[my_hosts]
VM-1 ansible_host=<VM-1_IP>
VM-2 ansible_host=<VM-2_IP>
```
2. Конфигурационный файл (ansible.cfg), в котором указал путь к файлу инвенторизации и другие необходимые настройки. Пример содержимого файла:
```php
[defaults]
inventory = inventory
```
3. Playbook (playbook.yml), в котором описал задачи, которые Ansible должен выполнить на управляемой машине (ВМ-2). Пример содержимого файла:
   
```yml
---
- name: Update packages and prepare VM-2
  hosts: VM-2
  tasks:
    - name: Update packages
      apt:
        upgrade: yes
        update_cache: yes

    - name: Install Docker dependencies
      apt:
        name: "{{ item }}"
        state: present
      loop:
        - apt-transport-https
        - ca-certificates
        - curl
        - software-properties-common

    - name: Add Docker GPG key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker repository
      apt_repository:
        repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable
        state: present

    - name: Install Docker
      apt:
        name: docker-ce
        state: present

    - name: Install Docker Compose
      pip:
        name: docker-compose
        state: present

    - name: Start Docker service
      service:
        name: docker
        state: started
```

В данном playbook выполняются следующие действия:

* Обновление пакетов на управляемой машине (ВМ-2).
* Установка зависимостей Docker.
* Добавление ключа GPG для репозитория Docker.
* Добавление репозитория Docker.
* Установка Docker и Docker Compose.
* Запуск службы Docker.

## Часть 3 (проверка работоспособности):

После выполнения основной части работы, я выполнил следующие действия:
1. Зашел на ВМ-2 по SSH для проверки статуса контейнеров и просмотра логов. Выполнил следующую команду в терминале ВМ-2:
```sh
$ docker ps -a
```
Получил список запущенных контейнеров и их статусы. Также просмотрел логи контейнеров с помощью команды `docker logs <container_id>`.

2. Отправил запрос к "серверному" контейнеру (из ЛР-7), чтобы проверить, что возвращается валидный response. Для этого воспользовался инструментом curl на ВМ-2:
```sh
$ curl http://localhost:<port>
```
Где `<port>` – порт, на котором запущен "серверный" контейнер.
