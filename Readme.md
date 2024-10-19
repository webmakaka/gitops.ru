# Исходники сайта [gitops.ru](https://gitops.ru)

<br/>

## Запустить локально с возможностью редактирования

Инсталлируете docker и docker-compose, далее:

```
$ cd ~
$ mkdir -p gitops.ru && cd gitops.ru
$ git clone --depth=1 https://github.com/webmakaka/gitops.ru.git .
$ docker-compose up
```

<br/>

Остается в браузере подключиться к localhost:80

<br/>

## (Устарело!)

<br/>

### Запустить gitops.ru на своем хосте с использованием docker контейнера:

```
$ docker run -i -t -p 80:80 --name gitops.ru marley/gitops.ru
```

<br/>

### Как сервис

```
$ sudo vi /etc/systemd/system/gitops.ru.service
```

вставить содержимое файла gitops.ru.service

```
$ sudo systemctl enable gitops.ru.service
$ sudo systemctl start  gitops.ru.service
$ sudo systemctl status gitops.ru.service
```

http://localhost:4006

<br/><br/>

---

<br/>

**Marley**

Any questions in english: <a href="https://gitops.ru/chat/">Telegram Chat</a>  
Любые вопросы на русском: <a href="https://gitops.ru/chat/">Телеграм чат</a>
