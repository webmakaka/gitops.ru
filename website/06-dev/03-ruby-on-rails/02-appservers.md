---
layout: page
title: Сервера приложений для Ruby on Rails проектов
description: Сервера приложений для Ruby on Rails проектов
keywords: dev, ruby on rails, Сервера приложений для Ruby on Rails проектов
permalink: /dev/ruby-on-rails/app-servers/
---

# Сервера приложений для Ruby on Rails проектов:

<br/>

### WebServer (WEBrick)

    $ rails server

http://localhost:3000/

Чтобы подключиться к WebRick с любого удаленного хоста:

    $ rails server -b 0.0.0.0 -p 3000

Можно еще использовать Unicorn и на передний план поставить Nginx.

А там уже кластеры, хуястеры при необходимости.
