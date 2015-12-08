
# Хост на Ubuntu

## DotPlant2

Документация по "DotPlant2": 
* <http://docs.dotplant.ru/ru/setup-example.html>
* <https://github.com/DevGroup-ru/dotplant2-docs>

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
$ php ../composer.phar install --prefer-dist --optimize-autoloader
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

###### Установка базовых настроек CMS:

```
$ ./installer
```


###### черновик
```
$ chmod 666 /home/admin/web/youfhe.ru/public_html/dotplant2/application/config/configurables-state/core.php
```

```
$ find /home/admin/web/youfhe.ru/public_html/dotplant2/application/config* -type d -exec chmod 666 {} \;

/home/admin/web/youfhe.ru/public_html/dotplant2/application/extensions/DefaultTheme/assets/theme/sass

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


## Настройка Nginx

```
$ sudo mkdir /etc/nginx/conf.d.src
$ sudo cp /etc/nginx/conf.d/188.166.17.183.conf /etc/nginx/conf.d.src/188.166.17.183.conf 
```

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
    listen 188.166.17.183:80 default;
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
$ sudo rm /etc/nginx/conf.d/188.166.17.183.conf
```

Перзапустим службы

```
$ sudo service nginx restart
$ sudo service php5-fpm restart
```

**Проверить**. Убедится что файл конфигурации `nginx.youfhe.ru.conf` попадает в backup.

**Проверено**. Файл конфигурации `nginx.youfhe.ru.conf` попадает в backup.


###### Группы


```
$ cat /etc/passwd
$ cat /etc/group
```

```
$ gpasswd -a nginx admin
$ gpasswd -a root admin
$ gpasswd -a www-data admin
```


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



###### Cron

```
$ su
$ crontab -e 
0 */6 * * * automysqlbackup /etc/automysqlbackup/myserver.conf
```

## VESTA

Для фунционирвания и доступа к ресурсам, процессе развертывания, были изменены настройки:

* /usr/local/vesta/conf/mysql.conf

## VESTA. Отключение резервного копирования

Заходим в задания cron и находим строку sudo /usr/local/vesta/bin/v-backup-users и переводим задание в статус "ЗАБЛОКИРОВАТЬ" ("SUSPEND").


## Ограничение доступа по SSH

В файле `/etc/ssh/sshd_config` вносим изменения:

```
PermitRootLogin yes
AllowUsers 87.117.167.118
```

```
$ service ssh restart
```


## Настройка DNS-записи

Информация: 
* <http://www.8host.com/blog/redirekt-domena-s-www-na-bez-www-na-nginx-v-centos-7/>

URL: <https://cloud.digitalocean.com/domains/youfhe.ru>

**Zone File** 

```
$ORIGIN youfhe.ru.
$TTL 1800
youfhe.ru. IN SOA ns1.digitalocean.com. hostmaster.youfhe.ru. 1449495774 10800 3600 604800 1800
youfhe.ru. 1800 IN NS ns1.digitalocean.com.
youfhe.ru. 1800 IN NS ns2.digitalocean.com.
youfhe.ru. 1800 IN NS ns3.digitalocean.com.
youfhe.ru. 1800 IN A 188.166.17.183
www.youfhe.ru. 1800 IN A 188.166.17.183
```


## Настройка почтового сервера Exim4

Информация:
* [http://help.ubuntu.ru/wiki/...](http://help.ubuntu.ru/wiki/%D1%80%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE_%D0%BF%D0%BE_ubuntu_server/%D0%BF%D0%BE%D1%87%D1%82%D0%BE%D0%B2%D1%8B%D0%B5_%D1%81%D0%B5%D1%80%D0%B2%D0%B8%D1%81%D1%8B/exim4) 

* http://lib.clodo.ru/email/exim_debian.html


```
$ sudo dpkg-reconfigure exim4-config
```

* send mail by smarthost; no local mail

* System mail name: youfhe.ru

* IP address to listen on for incomming SMTP connections: 127.0.0.1 ; ::1

* Other destinations for which mail accepted: h01.mylinker.ru

* Visible domain name for local users: h01.mylinker.ru

* IP address or host name of outgoing smarthost: h01.mylinker.ru

* Keep number of DNS-queries minimal (Dial-on-Demand)? : yes

* Split configuration info small files? : no


Все параметры, которые вы настроите в пользовательском интерфейсе будут сохранены в файле `/etc/exim4/update-exim4.conf`. Если вы захотите что-то перенастроить, или перезапустите мастера настройки или вручную поправьте данный файл любым редактором. После настройки вам потребуется выполнить следующую команду для создания главного файла настроек: 

```
$ sudo update-exim4.conf
```

```
$ sudo /etc/init.d/exim4 start
```



###### Аутентификация SMTP

Эта секция раскрывает как настроить Exim4 для использования SMTP-AUTH с TLS и SASL.

Первым шагом будет создание сертификата для использования TLS. Введите следующее в терминале: 

```
$ sudo /usr/share/doc/exim4-base/examples/exim-gencert
Country Code (two latter): RU
State or Province Name (full name): Tatarstan
Localy Name (eg, city): Kazan
Organization Name (eq, company; recommended): AK
Organization Unit name (eq, section):
Server name (eg. ssl.domain.tld; required!!!): youfhe.ru
Email Address: 
```

??? Теперь Exim4 нуждается в настройке TLS. Отредактируйте `/etc/exim4/conf.d/main/03_exim4-config_tlsoptions`, добавив следующее: 

```
MAIN_TLS_ENABLE = yes
```

###### Добавление пользователя

Чтобы внешний почтовый клиент имел возможность соединиться с вашим новым exim сервером, требуется добавить нового пользователя в exim, используя следующие команды:

```
$ sudo /usr/share/doc/exim4/examples/exim-adduser
```


```
$ sudo update-exim4.conf
$ sudo /etc/init.d/exim4 restart
```



## Exim4 и Dovecot

### Dovecot

Информация:

* http://vladimir-stupin.blogspot.ru/2014/05/exim-dovecot-sql.html

```
# groupadd -g 120 -r vmail
# useradd -g 120 -r -u 120 vmail
# mkdir /home/vmail
# chown vmail:vmail /home/vmail
# chmod u=rwx,g=rx,o= /home/vmail
```









## Настрйока почтовых уведомлений в DotPlant2

Просматривая конфигурационные фалй в папрке `/dotplant2/application/config` видим комментарии:

```
/*
 * ! WARNING !
 *
 * This file is auto-generated.
 * Please don't modify it by-hand or all your changes can be lost.
 */
```

Следовательно эти настройки надо ментя через админку. Путь к станице `/config/backend/index`. Изменяем данные в колонке `Конфигурации E-mail`:

**Замечание:** 

* `Mail transport: Mail` нужно заполнить только поле "Mail from".
* `Mail transport: SMTP` нужно заполнить поля:
  * Mail server,
  * Mail username,
  * Mail password, 
  * Mail server port, 
  * Mail encryption.


