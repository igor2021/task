

# DotPlant2

```
$ apt-get update
$ apt-get upgrade
```

```
$ apt-get install  nginx php5-fpm php5-gd php5-json mysql-server php5-mysql php5-cli \
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
$ cp /etc/php5/fpm/php.ini /etc/php5/fpm/php.ini.src
$ sudo vi /etc/php5/fpm/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```

```
$ cp /etc/php5/apache2/php.ini /etc/php5/apache2/php.ini.src
$ sudo vi /etc/php5/apache2/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```

```
$ cp /etc/php5/cli/php.ini /etc/php5/cli/php.ini.src
$ sudo vi /etc/php5/cli/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```

```
$ cp /etc/php5/cgi/php.ini /etc/php5/cgi/php.ini.src
$ sudo vi /etc/php5/cgi/php.ini
date.timezone = 'Europe/Moscow'
expose_php = Off
```



Настройка `nginx`:

```
$ vi /etc/nginx/conf.d/188.166.17.183.conf
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

# node npm

```
```






