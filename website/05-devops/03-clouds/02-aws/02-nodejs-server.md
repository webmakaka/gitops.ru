---
layout: page
title: Инсталляция Node.js в облаке AWS
description: Инсталляция Node.js в облаке AWS
keywords: Инсталляция Node.js в облаке AWS
permalink: /devops/clouds/aws/nodejs-server/
---

# Инсталляция Node.js в облаке AWS

Имеем Amazon'овский образ RedHat based

Подключились к серверу.

    $ sudo yum update -y
    $ sudo yum install -y gcc-c++ make
    $ sudo yum install -y openssl-devel
    $ sudo yum install -y git

<br/>

**Инсталляция node.js**

    $ cd /tmp/
    $ git clone git://github.com/joyent/node.git
    $ cd node
    $ ./configure
    $ make
    $ sudo make install

<br/>
    
    $ sudo su
    # vi /etc/sudoers

{% highlight text %}

Нужно добавить в secure_path :/usr/local/bin

Defaults secure_path = /sbin:/bin:/usr/sbin:/usr/bin
заменить на
Defaults secure_path = /sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin

{% endhighlight %}

**Инсталляция npm**

    $ cd /tmp/
    $ git clone https://github.com/isaacs/npm.git
    $ cd npm
    $ sudo make install

**Возможная проверка работы Node.js приложения в облаке AWS**

    $ vi server.js

{% highlight text %}

require("http").createServer(function(request, response){
response.writeHeader(200, {"Content-Type": "text/plain"});  
 response.write("Hello World!");  
 response.end();
}).listen(8080);

{% endhighlight %}

    $ node server.js
    $ curl http://localhost:8080

P.S. Чтобы работали приложения (в том числе сгенерированные с помощью express) в opsWorks, нужно чтобы стартовый скрипт находился в корне каталога приложения и имел имя server.js.  
При этом стартовать приложение должно на 80 порту.

Т.е. если приложение сконфигурировано с помощью express. Нужно /bin/www переименовать в ./server.js. Указать порт 80 и в package.json указать server.js в качестве скрипта для старта.

---

см:  
http://iconof.com/blog/how-to-install-setup-node-js-on-amazon-aws-ec2-complete-guide/  
http://stackoverflow.com/questions/10578249/hosting-nodejs-application-in-ec2
