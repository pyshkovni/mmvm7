# Установка MatterMost 7.0. на Ubuntu 20.04

Данный гайд по шагам описывает процесс запуска корпоративного мессенджера с открытым исходным кодом [MatterMost](https://mattermost.com) на виртуальной машине Ubuntu.

![MatterMost](~/../images/mmvm7.gif)

[Ссылка на официальную документацию](https://docs.mattermost.com/install/installing-ubuntu-2004-LTS.html)


## Генерация SSH-ключей
Для обеспечения безопасного удаленного доступа к серверу необходимо сгенерировать SSH-ключи.  
* Для Смартфона - воспользуйтесь [Termius](https://termius.com/)  
* Для MacOS - откройте терминал 
* Для Windows 10/11 - откройте PowerShell
Ваедите в консоли
    >>> mkdir /Users/<пользователь>/.sshkeys/mmvm7/

    >>> ssh-keygen
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Users/pyshkovni/.ssh/id_rsa): 
    --> укажи путь /Users/<пользователь>/.sshkeys/id_rsa --> жми Return
    Enter passphrase (empty for no passphrase): --> введи n12345 (пример)



## Создание виртуальной машины
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

## Доменное имя
Купить доменное имя (например, в reg.ru).

## Подключение к ВМ

В терминале введите команду ssh _\<login>@\<public ip>_

## Установка программного обеспечения

    sudo -i
    
    apt update

    apt upgrade

    apt install mysql-server

    service mysql status

    mysql

    create user 'mmuser'@'%' identified by '<ваш_пароль>';

    create database mattermost;

    grant all privileges on mattermost.* to 'mmuser'@'%';

    GRANT ALTER, CREATE, DELETE, DROP, INDEX, INSERT, SELECT, UPDATE, REFERENCES ON mattermost.* TO 'mmuser'@'%';
    
    exit

    wget https://releases.mattermost.com/7.4.0/mattermost-7.4.0-linux-amd64.tar.gz

    tar -xvzf mattermost*.gz

    mv mattermost /opt

    mkdir /opt/mattermost/data

    useradd --system --user-group mattermost

    chown -R mattermost:mattermost /opt/mattermost

    chmod -R g+w /opt/mattermost

    vim /opt/mattermost/config/config.json

    =================================================================
    "DriverName": "mysql",
    "DataSource": "mmuser:<ваш пароль>@tcp(<host-name-or-IP>:3306)/mattermost?charset=utf8mb4,utf8&writeTimeout=30s"

    "SiteURL": "http://www.matterchat.ru",
    
    :wq
    =================================================================

    cd /opt/mattermost

    sudo -u mattermost ./bin/mattermost

    http://84.201.161.192:8065

    Ctrl C

    touch /lib/systemd/system/mattermost.service

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


    systemctl daemon-reload

    systemctl status mattermost.service

    systemctl start mattermost.service

    systemctl enable mattermost.service

    http://www.matterchat.ru:8065









