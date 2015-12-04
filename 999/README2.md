

# DotPlant2

```
$ apt-get update
$ apt-get upgrade
```

```
$ apt-get install  nginx php5-fpm php5-gd php5-json mysql-server php5-mysql php5-cli \
	php5-memcached memcached php5-curl php5-intl git
```

......


```
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
        include /etc/nginx/fastcgi_params;
    }

    location ~ /\.ht {
       deny all;
    }
}
```



```
$ cd /usr/local/vesta/data/templates/web/nginx/php5-fpm/
$ cp ./deafult.tpl ../php5-fpm.tpl
$ cp ./deafult.stpl ../php5-fpm.stpl
$ cd ..
$ chmod 755 deafult.tpl
$ chmod 755 deafult.stpl 
```



server {
    listen      188.166.17.183:8080;
    
    server_name youfhe.ru www.youfhe.ru;
    
    root        /home/admin/web/youfhe.ru/public_html/dotplant2/application/web;
    index       index.php index.html index.htm;
    
    access_log  /var/log/nginx/domains/youfhe.ru.log combined;
    access_log  /var/log/nginx/domains/youfhe.ru.bytes bytes;
    error_log   /var/log/nginx/domains/youfhe.ru.error.log error;

    location / {
		
        location ~* ^.+\.(jpeg|jpg|png|gif|bmp|ico|svg|css|js)$ {
            expires     max;
        }
		
		try_files $uri $uri/ /index.php?$args;
		
        location ~ [^/]\.php(/|$) {
            try_files $uri =404;			
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass    unix:/var/run/php5-fpm.sock;
            fastcgi_index   index.php;
            include         /etc/nginx/fastcgi_params;
        }
    }

    location ~ /\.ht {
       deny all;
    }
}













server {
    listen      188.166.17.183:8080;
    
    server_name youfhe.ru www.youfhe.ru;
    
    root        /home/admin/web/youfhe.ru/public_html/dotplant2/application/web;
    index       index.php index.html index.htm;
    
    access_log  /var/log/nginx/domains/youfhe.ru.log combined;
    access_log  /var/log/nginx/domains/youfhe.ru.bytes bytes;
    error_log   /var/log/nginx/domains/youfhe.ru.error.log error;

    location / {

        location ~* ^.+\.(jpeg|jpg|png|gif|bmp|ico|svg|css|js)$ {
            expires     max;
        }

        location ~ [^/]\.php(/|$) {
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            if (!-f $document_root$fastcgi_script_name) {
                return  404;
            }

            fastcgi_pass    unix:/var/run/php5-fpm.sock;
            fastcgi_index   index.php;
            include         /etc/nginx/fastcgi_params;
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

    include     /etc/nginx/conf.d/phpmyadmin.inc*;
    include     /etc/nginx/conf.d/phppgadmin.inc*;
    include     /etc/nginx/conf.d/webmail.inc*;

    include     /home/admin/conf/web/nginx.youfhe.ru.conf*;
}



