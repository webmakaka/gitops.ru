# Исходники сайта [gitops.ru](https://gitops.ru)

<br/>

### Запустить локально с возможностью редактирования

Инсталлируете docker и docker-compose, далее:

    $ cd ~
    $ mkdir -p gitops.ru && cd gitops.ru
    $ git clone --depth=1 https://github.com/webmakaka/gitops.ru.git .
    $ docker-compose up

<br/>

Остается в браузере подключиться к localhost:80


<br/>

### Запустить gitops.ru на своем хосте с использованием docker контейнера:

    $ docker run -i -t -p 80:80 --name gitops.ru marley/gitops.ru

<br/>

### Как сервис

    # vi /etc/systemd/system/gitops.ru.service

вставить содержимое файла gitops.ru.service

    # systemctl enable gitops.ru.service
    # systemctl start  gitops.ru.service
    # systemctl status gitops.ru.service

http://localhost:4006
