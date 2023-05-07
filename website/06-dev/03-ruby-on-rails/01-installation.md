---
layout: page
title: Инсталляция Ruby on Rails в Centos 6.X
description: Инсталляция Ruby on Rails в Centos 6.X
keywords: dev, ruby on rails, Инсталляция Ruby on Rails в Centos 6.X
permalink: /dev/ruby-on-rails/installation/centos/6.X/
---

# Инсталляция Ruby on Rails в Centos

Пишу то к чему пришел. Из разных вариантов инсталляции, данный мне видится более предпочтительным. Впрочем я не особо занимаюсь этой технологией.

Имеется также вариант, который можно посмотреть на сайте rvm.io

<br/>

### С использованием rbenv

    # yum install -y \
    which \
    tar \
    curl \
    openssl-devel \
    git \
    gcc

Создаю пользователя developer.

    # useradd \
    -d /home/developer \
    -m developer

<br/>

    # passwd developer

<br/>

    # mkdir /rails_projects
    # chown -R developer /rails_projects/

<br/>

    # su - developer

<br/>

    $ cd ~/
    $ git clone git://github.com/sstephenson/rbenv.git .rbenv
    $ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
    $ echo 'eval "$(rbenv init -)"' >> ~/.bashrc
    $ exec $SHELL

<br/>

    $ git clone git://github.com/sstephenson/ruby-build.git ~/.rbenv/plugins/ruby-build
    $ echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
    $ exec $SHELL

<br/>

    $ rbenv install --list

<br/>

    $ rbenv install 2.2.3

<br/>

    $ rbenv versions
    2.2.3

<br/>

    $ rbenv global 2.2.3
    $ ruby -v
    ruby 2.2.3p173 (2015-08-18 revision 51636) [x86_64-linux]

<br/>

### RubyGems

    $ gem -v
    2.4.5.1

<br/>

    $ gem list

<br/>

    $ gem update --system

<br/>

### Rails

    $ echo "gem: --no-ri --no-rdoc" > ~/.gemrc

<br/>

    $ gem install bundler
    $ rbenv rehash
    $ bundle -v
    Bundler version 1.11.2

<br/>

    $ gem install rails --no-ri --no-rdoc
    $ rbenv rehash
    $ rails -v
    Rails 4.2.5
