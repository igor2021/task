
# Хост на Ubuntu

## DotPlant2

Документация по "DotPlant2": <http://docs.dotplant.ru/ru/setup-example.html>

###### Обновляем пакеты и устанавливаем необходимые пакеты:

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

```
$ apt-get install nginx php5-fpm php5-gd php5-json mysql-server php5-mysql php5-cli \
	php5-memcached memcached php5-curl php5-intl git
```


###### Создадим базу MySQL:

```
# mysql -u root -p
CREATE DATABASE dotplant2 DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'dotplant2'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON dotplant2.* TO 'dotplant2'@'localhost';
FLUSH PRIVILEGES;
```

###### Склонируем гит репозиторий и обновим зависимости:

```
$ cd ~/web/youfhe.ru/public_html
$ git clone https://github.com/DevGroup-ru/dotplant2.git
$ cd dotplant2/application
$ php ../composer.phar global require "fxp/composer-asset-plugin:~1.0"
# php ../composer.phar install --prefer-dist --optimize-autoloader
```

###### Проверяем тербования:

```
$ php requirements.php
```

###### Устанавливаем недостающие пакеты, правим файлы настроек:

```
$ sudo apt-cache search php5 imagick
$ sudo apt-get install php5-imagick
$ sudo apt-cache search php5 apc
$ sudo apt-get install php5-apcu
```

```
$ sudo cp /etc/php5/fpm/php.ini /etc/php5/fpm/php.ini.src
$ sudo vi /etc/php5/fpm/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```

```
$ sudo cp /etc/php5/apache2/php.ini /etc/php5/apache2/php.ini.src
$ sudo vi /etc/php5/apache2/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```

```
$ sudo cp /etc/php5/cli/php.ini /etc/php5/cli/php.ini.src
$ sudo vi /etc/php5/cli/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```

```
$ sudo cp /etc/php5/cgi/php.ini /etc/php5/cgi/php.ini.src
$ sudo vi /etc/php5/cgi/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```


###### Настройка Nginx

```
$ sudo vi /etc/nginx/conf.d/188.166.17.183.conf
server {
    listen 188.166.17.183:80 default;
	
	server_name youfhe.ru www.youfhe.ru;

    root /home/admin/web/youfhe.ru/public_html/dotplant2/application/web;
    index index.php;

    access_log  /var/log/nginx/domains/youfhe.ru.log combined;
    access_log  /var/log/nginx/domains/youfhe.ru.bytes bytes;
    error_log   /var/log/nginx/domains/youfhe.ru.error.log error;

    location / {    
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;    
        include fastcgi_params;
        <b color="red">fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;<b>  
    }

    location ~ /\.ht {
       deny all;
    }
}
```

```
$ sudo service nginx restart
$ sudo service php5-fpm restart
```


<b color="red">PS.</b> `Vesta` по умолчанию читает логи `apache2`. Вопрос (или я чего-то незнаю): а какже вывести их через `Vesta`?


###### Установка базовых настроек CMS:

```
$ ./installer
```

## Правки по DotPlant2

###### Cоздание новой базы MySQL, дамп и востанновление из дампа, удаление старой базы:

```
$ mysql -u root -p
CREATE DATABASE youfh DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'youfh'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON youfh.* TO 'youfh'@'localhost';
FLUSH PRIVILEGES;
```

```
$ mysqldump -u root -p dotplant2 > /home/admin/tmp/dotplant2.sql 
$ mysql -u root -p youfh < /home/admin/tmp/dotplant2.sql 
```

```
$ mysql -u root -p
$ DROP DATABASE `dotplant2`;
```

Также правим в настройках DotPlant2 `./config/db-local.php`.


## Composer

```
$ curl -sS https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/local/bin/composer
$ composer -V
```

## Git

```
$ dpkg -s git | grep Status
$ sudo apt-get install git
```

## PostgerSQL

```
$ dpkg -s postgresql | grep Status
$ sudo apt-get install postgresql postgresql-contrib phppgadmin
Setting up phppgadmin (5.1-1) ...
 * Reloading web server apache2         * 
 * Apache2 is not running
$ service apache2 start
$ sudo apt-get install postgresql postgresql-contrib phppgadmin
```


###### Интеграция с VESTA

Документация по установке в "VESTA":  <http://vestacp.com/docs/> (How to set up PostgreSQL on a Debian or Ubuntu).

```
$ sudo cp /etc/postgresql/*/main/pg_hba.conf /etc/postgresql/*/main/pg_hba.conf.src
$ sudo wget http://c.vestacp.com/0.9.8/debian/pg_hba.conf -O /etc/postgresql/*/main/pg_hba.conf
```

```
$ sudo service postgresql restart
```

```
$ su - postgres
psql -c "ALTER USER postgres WITH PASSWORD '<password>'"
exit
```

```
$ vi /usr/local/vesta/conf/vesta.conf
...
DB_SYSTEM='mysql,pgsql'
```

```
$ v-add-database-host pgsql localhost postgres <password>
```

```
$ cp /etc/phppgadmin/config.inc.php /etc/phppgadmin/config.inc.php.src
$ wget http://c.vestacp.com/0.9.8/debian/pga.conf -O /etc/phppgadmin/config.inc.php
$ mkdir /etc/apache2/conf.d.src
$ cp /etc/apache2/conf.d/phppgadmin /etc/apache2/conf.d.src/phppgadmin
$ wget http://c.vestacp.com/0.9.8/debian/apache2-pga.conf -O /etc/apache2/conf.d/phppgadmin
```

```
$ service apache2 restart
```

## sysv-rc-conf

Пакет для управления автозагрузкой служб.

```
$ sudo apt-get install sysv-rc-conf
```

Проверить режимы автозагрузки можно командой:

```
$ sysv-rc-conf --list
```


## Node npm

```
$ dpkg -s nodejs | grep Status
$ sudo apt-cache search nodejs npm
$ sudo apt-get install npm
$ nodejs -v
$ npm -v
```

## Redis, redis-desktop-manager


###### redis

Документация по service redis:

  * http://softtime.info/view/Redis
  

```
$ dpkg -s redis | grep Status
$ sudo apt-cache search redis
$ sudo apt-get install redis-server
```

```
$ sudo service redis-server status
$ sudo service redis-server start
$ sysv-rc-conf --list redis-server
```




###### redis-desktop-manager

Документация по "RedisDesktopManager": <https://github.com/uglide/RedisDesktopManager/wiki>  
**Лучше менеджер установить на удаленной машине и подлючаться по SSH.**

```
$ cd ~/tmp
$ wget https://github.com/uglide/RedisDesktopManager/releases/download/0.8.3/redis-desktop-manager_0.8.3-120_amd64.deb
$ 
```

Документация о том как собрать из исходников: <https://github.com/uglide/RedisDesktopManager/wiki/Build-from-source>






