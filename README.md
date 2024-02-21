# Пошаговая установка MatterMost 7.4. на Ubuntu 20.04

Данный гайд по шагам описывает процесс запуска корпоративного мессенджера с открытым исходным кодом [MatterMost](https://mattermost.com) на виртуальной машине Ubuntu.

![MatterMost](~/../images/mmvm7.gif)

[Ссылка на официальную документацию](https://docs.mattermost.com/install/installing-ubuntu-2004-LTS.html)


## 1. Генерация SSH-ключей
Для обеспечения безопасного удаленного доступа к серверу необходимо сгенерировать SSH-ключи.  
* Для Смартфона - воспользуйтесь [Termius](https://termius.com/)  
* Для MacOS - откройте терминал 
* Для Windows 10/11 - откройте PowerShell

Введите в консоли команду создания папки для ключей

    mkdir -p /Users/<пользователь>/.sshkeys/mmvm7/

Сгенерируйте ключи по указанной директории

    ssh-keygen

Вывод консоли на команду ssh-keygen

    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/pyshkovni/.ssh/id_rsa): 
    --> укажите путь /Users/<пользователь>/.sshkeys/mmvm7/id_rsa --> нажмите Return
    Enter passphrase (empty for no passphrase): --> введите n12345 (пример)

## 2. Создание виртуальной машины
Виртуальная машина создавалась в облачном сервисе [Yandex Cloud](https://cloud.yandex.ru). Виртуальную машину можно создать в рамках бесплатного гранда от Yandex Cloud. Для этого зарегистрируйтесь в сервисе и прикрепите платежную карту.

1. Перейдите в сервисах в Compute Cloud
2. Создайте VM со следующими параметрами:
   * Ubuntu 20.04 _(данная версия обязательна)_
   * Загрузочный диск SSD - 15 Гб _(затем можно добавить дополнительный диск для хранения данных)_
   * Платформа Intel Icelake
   * vCPU 2 ядра
   * RAM 4 Гб 
   * Введите логин
   * Вставьте скопированный ssh-ключ
   * Включите доступ к серийной консоли
3. Нажмите создать

## 3. Доменное имя
Купить доменное имя без хостинга (например, в [reg.ru](https://www.reg.ru)).

## 4. Подключение к ВМ и подготовка к настройке

В терминале введите команду 

    ssh -i /Users/<пользователь>/.sshkeys/mmvm7/ <login>@<public ip>

Перейдите в пользователя root

    sudo -i

Обновите ссылки репозитории

    apt update

Обновите пакеты

    apt upgrade

## 5. Установка MySQL
MySQL - это система управления базам данных MySQL  
Введите команду, чтобы установить пакет

    apt install mysql-server

Проверьте статус сервиса. Должно быть _`active`_

    service mysql status

Зайдите в MySQL

    mysql

Появится приветствие и командная строка SQL

### Настройка БД
Создайте пользователя mmuser

    create user 'mmuser'@'%' identified by '<ваш_пароль>';

Создайте базу данных mattermost

    create database mattermost;

Определите для него полные права

    grant all privileges on mattermost.* to 'mmuser'@'%';

Предоставьте доступ ко всем командам

    GRANT ALTER, CREATE, DELETE, DROP, INDEX, INSERT, SELECT, UPDATE, REFERENCES ON mattermost.* TO 'mmuser'@'%';
    
Закройте MySQL

    exit

## 6. Установка mattermost
Скачайте архив с программой по ссылке (версия mattermost 7.4.0)

    wget https://releases.mattermost.com/7.4.0/mattermost-7.4.0-linux-amd64.tar.gz

Распакуйте архив

    tar -xvzf mattermost*.gz

Перенесите распакованную папку mattermost в папку /opt

    mv mattermost /opt

Создайте папку /opt/mattermost/data

    mkdir /opt/mattermost/data

## 7. Права пользователя mattermost на ВМ
Создайте пользователя mattermost

    useradd --system --user-group mattermost

Задайте владельца каталога /opt/mattermost 

    chown -R mattermost:mattermost /opt/mattermost

Определите права владения каталогом /opt/mattermost
    
    chmod -R g+w /opt/mattermost

## 8. Настройка конфигурации mattermost
Откройте файл config.json в редакторе vim
 
    vim /opt/mattermost/config/config.json

С помощью команды поиска в vim (нажмите /) найдите и отредактируйте следующие строки.  
Проверьте наличие символа _&_ между _utf8_ и _writeTimeout_!!! 

    =================================================================
    ...
    "DriverName": "mysql",
    "DataSource": "mmuser:<ваш пароль>@tcp(localhost:3306)/mattermost?charset=utf8mb4,utf8&writeTimeout=30s",
    ...
    =================================================================

Найдите строку SiteURL и добавьте доменное имя. Сохраните файл и закройте его с помощью :wq

    =================================================================
    ...
    "SiteURL": "http://www.<имя>.<домен>",
    ...
    
    :wq
    =================================================================

Выйдите из пользователя root

    exit

## 9. Запуск и проверка в работоспособности
Перейдите в папку /opt/mattermost

    cd /opt/mattermost

Запустите mattermost

    sudo -u mattermost ./bin/mattermost

Откройте браузер и перейдите по адресу

    http://<Публичный IP-адрес>:8065

По завершению проверки вернитесь терминал и прекратите процесс

    Нажмите сочетание клавиш Ctrl C

## 10. Запуск в фоновом режиме
Для запуска в фоновом режиме необходимо прописать её в systemd  
Перейдите в пользователя root

    sudo -i

Создайте файл mattermost.service

    touch /lib/systemd/system/mattermost.service

Откройте файл в vim

    vim /lib/systemd/system/mattermost.service

Скопируйте настройки ниже и вставьте их в файл

    [Unit]
    Description=Mattermost
    After=network.target
    After=mysql.service
    BindsTo=mysql.service
    
    [Service]
    Type=notify
    ExecStart=/opt/mattermost/bin/mattermost
    TimeoutStartSec=3600
    KillMode=mixed
    Restart=always
    RestartSec=10
    WorkingDirectory=/opt/mattermost
    User=mattermost
    Group=mattermost
    LimitNOFILE=49152
    
    [Install]
      WantedBy=mysql.service


Перезапустите systemd

    systemctl daemon-reload

Проверьте статус mattermost.service. Должен быть _dead_

    systemctl status mattermost.service

Запустите mattermost.service

    systemctl start mattermost.service

Установите автоматический запуск mattermost.service при запуске ВМ

    systemctl enable mattermost.service


## 11. Настройка пользователей mattermost
В браузере перейдите по домену в mattermost либо воспользуйтесь публичным ip-адресом

    http://www.<имя>.<домен>:8065
    http://<Публичный IP-адрес>:8065

____
_раздел в разработке_
<br>
<br>
## Проблемы
_раздел в разработке_
<br>
<br>
## Настройка сертификата
_раздел в разработке_


## Альтернативный запуск MatterMost через Docker.
Docker используется для упрощения процесса разработки, тестирования и развертывания приложений. Он позволяет разработчикам упаковывать свое приложение и все необходимые зависимости в один компактный образ, который можно легко переносить между разными средами и операционными системами. Образы Docker могут быть использованы многократно, что снижает затраты на разработку и тестирование.

### Команды для запуска
[Оффициальная документация](https://docs.mattermost.com/install/install-docker.html#install-docker)<br>
Подготовить виртуальную машину к работе с docker и docker-compose.

    sudo apt update
    sudo apt install docker.io
    sudo apt install docker-compose

Запустить docker в фоновом режиме

    sudo systemctl start docker

Создать группу пользователей docker

    sudo groupadd docker

Добавить текущего пользователя в группу docker

    sudo usermod -aG docker ${USER}

Как только Docker и пользователи будут настроены, запустите следующую команду:

    docker run --name mattermost-preview -d --publish 8065:8065 mattermost/mattermost-preview

Запускаем клонирование репозитория git

    git clone https://github.com/mattermost/docker && cd docker

Скопируем файл конфигурации

    cp env.example .env

Заходим в скопированный файл и редактируем его

    nano env.example .env

В строчке DOMAIN меняем значение `mm.example.com` на публичный ip-адрес виртуальной машины. Делаем это дважды!<br>
Настраиваем нужные директории

    mkdir -p ./volumes/app/mattermost/{config,data,logs,plugins,client/plugins,bleve-indexes} && sudo chown -R 2000:2000 ./volumes/app/mattermost

Запуск сборка образа mattermost через docker-compose (без NGINX)

    sudo docker-compose -f docker-compose.yml -f docker-compose.without-nginx.yml up -d

В браузере перейдите по ссылке `http://<публичный ip-адрес>:8065`.<br>
Процесс запуска с docker завершен!


    
