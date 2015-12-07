
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
$ mysql -u root -p
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
expose_php = Off
```

```
$ sudo cp /etc/php5/cli/php.ini /etc/php5/cli/php.ini.src
$ sudo vi /etc/php5/cli/php.ini
expose_php = Off
```

```
$ sudo cp /etc/php5/cgi/php.ini /etc/php5/cgi/php.ini.src
$ sudo vi /etc/php5/cgi/php.ini
expose_php = Off
```

###### Настройка Nginx

```
$ sudo mkdir /etc/nginx/conf.d.src
$ sudo cp /etc/nginx/conf.d/<host_or_ip>.conf /etc/nginx/conf.d.src/<host_or_ip>.conf 
```

```
$ sudo vi /etc/nginx/conf.d/<host_or_ip>.conf
server {
    listen <host_or_ip>:80 default;
	
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
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
       deny all;
    }
}
```

###### Настройка Nginx в Vesta

Нам нужно чтобы конфиги лежали внутри `Vesta`.

```
$ touch /home/admin/conf/web/nginx.youfhe.ru.conf
```

```
$ vi /home/admin/conf/web/nginx.youfhe.ru.conf
server {
    listen <host_or_ip>:80 default;
    server_name youfhe.ru;

    root /home/admin/web/youfhe.ru/public_html/dotplant2/application/web;
    index index.php;

    #access_log  /var/log/nginx/domains/youfhe.ru.log combined;
    #access_log  /var/log/nginx/domains/youfhe.ru.bytes bytes;
    #error_log   /var/log/nginx/domains/youfhe.ru.error.log error;

    access_log  /var/log/apache2/domains/youfhe.ru.log combined;
    access_log  /var/log/apache2/domains/youfhe.ru.bytes bytes;
    error_log   /var/log/apache2/domains/youfhe.ru.error.log error;

    location / {
        try_files $uri $uri/ /index.php?$args;

        location ~* ^.+\.(jpeg|jpg|png|gif|bmp|ico|svg|css|js)$ {
            expires max;
        }

        location ~ \.php$ {
            try_files $uri =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php5-fpm.sock;
            fastcgi_index index.php;    
            include fastcgi_params;
            fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        }

        location ~ /\.ht {
            deny all;
        }
    } 
    
    error_page  403 /error/404.html;
    error_page  404 /error/404.html;
    error_page  500 502 503 504 /error/50x.html;

    location /error/ {
        alias   /home/admin/web/youfhe.ru/document_errors/;
    }
    
    location ~* "/\.(htaccess|htpasswd)$" {
        deny    all;
        return  404;
    }
}
```

```
$ vi /home/admin/conf/web/nginx.conf
include /home/admin/conf/web/nginx.youfhe.ru.conf;
......
```

```
$ chown root.admin /home/admin/conf/web/nginx.youfhe.ru.conf
```

Удалим предыдущий файл конфигурации:

```
$ sudo rm /etc/nginx/conf.d/<host_or_ip>.conf
```

Перзапустим службы

```
$ sudo service nginx restart
$ sudo service php5-fpm restart
```

**TODO**: Убедится что файл конфигурации `nginx.youfhe.ru.conf` попадает в backup.


###### Настройка редиректа Nginx

Редирект с www на без www

```
$ sudo vi /etc/nginx/conf.d/redirect.conf
server {
     listen 80;
     server_name  www.youfhe.ru;
     rewrite ^ http://youfhe.ru$request_uri? permanent; 
}
```

###### Настройка DNS-записи

Информация взята с сайта: <http://www.8host.com/blog/redirekt-domena-s-www-na-bez-www-na-nginx-v-centos-7/>

Чтобы настроить редирект с www.example.com на example.com (или наоборот), нужно создать запись для каждого имени.

Откройте панель управления DNS.

Если записи домена (также называется зоной) на данный момент не существует, создайте её сейчас. В hostname укажите доменное имя (к примеру, example.com), в поле IP address нужно указать внешний IP-адрес сервера Nginx. Некоторые системы создают запись A, которая указывает на заданный IP-адрес, автоматически, а некоторые требуют создавать такие записи вручную.

Затем создайте еще одну запись А, на этот раз для адреса с префиксом www, указав тот же IP-адрес.


###### Установка базовых настроек CMS:

```
$ ./installer
```

## Правки по DotPlant2 (1)

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


## Правки по DotPlant2 (2)

Чтобы все-таки привязать базу к `VESTA` сделаем следующие правки.

###### Cоздание новой базы MySQL, дамп и востанновление из дампа, удаление старой базы:

```
$ mysql -u root -p
CREATE DATABASE admin_youfhe DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'admin_youfhe'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON admin_youfhe.* TO 'admin_youfhe'@'localhost';
FLUSH PRIVILEGES;
```

```
$ mysqldump -u root -p youfh > /home/admin/tmp/youfh.sql 
$ mysql -u root -p admin_youfhe < /home/admin/tmp/youfh.sql 
```

```
$ mysql -u root -p
$ DROP DATABASE `youfh`;
```

Также правим в настройках DotPlant2 `/config/db-local.php`.

Добавим базу `admin_youfhe` через `VESTA`.

## Отключить резервное копирование в VESTA.

Заходим в задания cron и находим строку sudo /usr/local/vesta/bin/v-backup-users и переводим задание в статус SUSPEND.

## Apache

Отключим `apache2` и его автозагрузку:

```
$ sudo service apache2 stop
  * Stopping web server apache2
$ sudo service apache2 status
  * apache2 is not running
```

```
$ sudo sysv-rc-conf apache2 off
$ sysv-rc-conf --list apache2
```


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

###### redis command line interface

```
$ redis-cli
QUIT
```

###### redis-desktop-manager

Документация по "RedisDesktopManager": <https://github.com/uglide/RedisDesktopManager/wiki>  
**Лучше менеджер установить на удаленной машине и подлючаться по SSH.**

```
$ cd ~/tmp
$ wget https://github.com/uglide/RedisDesktopManager/releases/download/0.8.3/redis-desktop-manager_0.8.3-120_amd64.deb
$ dpkg -i redis-desktop-manager_0.8.3-120_amd64.deb
```

Варианты запуска:
```
$ /usr/share/redis-desktop-manager/bin/rdm
```
```
$ redis-desktop-manager
```

Документация о том как собрать из исходников: <https://github.com/uglide/RedisDesktopManager/wiki/Build-from-source>


## AutoMySQLBackup

Ссылка для скачивания: <http://sourceforge.net/projects/automysqlbackup/>

```
$ cd ~/tmp
$ sudo cp ./automysqlbackup-v3.0_rc6.tar.gz /usr/local/src/
$ cd /usr/local/src
$ sudo mkdir automysqlbackup
$ sudo tar xvf automysqlbackup-v3.0_rc6.tar.gz -C automysqlbackup/
$ cd automysqlbackup
$ sudo ./install.sh
```

```
$ sudo mkdir /var/www/archive/
```

Находим и правим в файле `/etc/automysqlbackup/myserver.conf` следующие данные:

```
$ sudo vi /etc/automysqlbackup/myserver.conf
CONFIG_mysql_dump_username='backup'
CONFIG_mysql_dump_password='<password>'
CONFIG_mysql_dump_host='localhost'
CONFIG_backup_dir='/var/www/archive/'
CONFIG_db_names=()
CONFIG_db_exclude=( 'information_schema' 'performance_schema' )
CONFIG_table_exclude=( 'mysql.event' )
CONFIG_rotation_daily=6
CONFIG_rotation_weekly=35
CONFIG_rotation_monthly=150
```

Создаем пользователя `backup` базы данных `mysql`:

```
$ mysql -u root -p
CREATE USER 'backup'@'localhost' IDENTIFIED BY '<password>';
GRANT SELECT, LOCK TABLES ON *.* TO 'backup'@'localhost';
FLUSH PRIVILEGES;
QUIT;
```

Проверим запуск `automysqlbackup` с нашими параметрами:

```
$ su
$ automysqlbackup /etc/automysqlbackup/myserver.conf
$ chown root.root /var/www/archive* -R
   find /var/www/archive* -type f -exec chmod 400 {} \;
   find /var/www/archive* -type d -exec chmod 700 {} \;
```



###### cron

```
$ su
$ crontab -e 
0 */6 * * * automysqlbackup /etc/automysqlbackup/myserver.conf
```

## VESTA

Для фунционирвания и доступа к ресурсам, процессе развертывания, были изменены настройки:

* /usr/local/vesta/conf/mysql.conf


## Ограничение доступа по SSH

Это последнее что надо сдлетать. Но сначало надо убедится что все сделано как надо.

**TODO** 


