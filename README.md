

# DotPlant2

Документация: [http://docs.dotplant.ru/ru/setup-example.html](http://docs.dotplant.ru/ru/setup-example.html)

Обновляем пакеты и устанавливаем необходимые пакеты:

```
$ sudo apt-get update
$ sudo apt-get upgrade
```

```
$ apt-get install nginx php5-fpm php5-gd php5-json mysql-server php5-mysql php5-cli \
	php5-memcached memcached php5-curl php5-intl git
```

Создадим базу:

```
$ mysql -u root -p
CREATE DATABASE dotplant2 DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'dotplant2'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON dotplant2.* TO 'dotplant2'@'localhost';
FLUSH PRIVILEGES;
```

Склонируем гит репозиторий и обновим зависимости:

```
$ cd ~/web/youfhe.ru/public_html
$ git clone https://github.com/DevGroup-ru/dotplant2.git
$ cd dotplant2/application
$ php ../composer.phar global require "fxp/composer-asset-plugin:~1.0"
# php ../composer.phar install --prefer-dist --optimize-autoloader
```

Проверяем тербования:

```
$ php requirements.php
```

Устанавливаем недостающие пакеты, правим файлы настроек:

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



Настройка `nginx`:

```
$ sudo vi /etc/nginx/conf.d/188.166.17.183.conf
server {
    listen 188.166.17.183:80 default;

    root /home/admin/web/youfhe.ru/public_html/dotplant2/application/web;
    index index.php;

    server_name youfhe.ru www.youfhe.ru;

    location / {    
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
    #root /home/admin/web/youfhe.ru/public_html/dotplant2/application/web;
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        #fastcgi_index index.php;    
        include fastcgi_params;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;  
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

Установка базовых настроек CMS:

```
$ ./installer
```

# Composer

```
$ curl -sS https://getcomposer.org/installer | php
$ sudo mv composer.phar /usr/local/bin/composer
$ composer -V
```

# Git

```
$ dpkg -s git | grep Status
$ sudo apt-get install git
```

# PostgerSQL

```
$ dpkg -s postgresql | grep Status
$ sudo apt-get install postgresql postgresql-contrib phppgadmin
Setting up phppgadmin (5.1-1) ...
 * Reloading web server apache2         * 
 * Apache2 is not running
$ service apache2 start
$ sudo apt-get install postgresql postgresql-contrib phppgadmin
```

Документация по установке в "VESTA":  [http://vestacp.com/docs/](http://vestacp.com/docs/) (How to set up PostgreSQL on a Debian or Ubuntu).

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



# Node npm

```
$ dpkg -s nodejs | grep Status
$ sudo apt-cache search nodejs npm
$ sudo apt-get install npm
```

# Redis

TODO



