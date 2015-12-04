

# DotPlant2

```
$ apt-get update
$ apt-get upgrade
```

```
$ apt-get install  nginx php5-fpm php5-gd php5-json mysql-server php5-mysql php5-cli \
	php5-memcached memcached php5-curl php5-intl git
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






