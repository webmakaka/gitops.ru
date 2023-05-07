---
layout: page
title: Подключиться к серверу в облаке AWS в командной строке linux
description: Подключиться к серверу в облаке AWS в командной строке linux
keywords: Подключиться к серверу в облаке AWS в командной строке linux
permalink: /devops/clouds/aws/connect-to-server/
---

# Подключиться к серверу в облаке AWS в командной строке linux

Создать Key Pair в консоли AWS и с скачать ключ.

<br/>

    $ chmod 400 /home/marley/Downloads/AWS-Key.pem
    $ ssh -i /home/marley/Downloads/AWS-Key.pem ec2-user@<ip_сервера>.

<br/>

см:  
http://www.youtube.com/watch?v=Ix5IDuyamuY

<br/>

### Converting a ppk file to a pem file for accessing AWS ec2 instances on Linux

    $ sudo apt-get install putty-tools

    $ puttygen ppkkey.ppk -O private-openssh -o pemkey.pem

http://webkul.com/blog/convert-a-ppk-file-to-a-pem-file/
