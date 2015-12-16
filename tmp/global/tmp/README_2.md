# youfhe


## База youfhe

```
$ mysql -u root -p
CREATE DATABASE youfhe DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'youfhe'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON youfhe.* TO 'youfhe'@'localhost';
FLUSH PRIVILEGES;
``` 

```
$ gunzip < /home/admin/tmp/youfhe_2015-12-08_06h25m.Tuesday.sql.gz | mysql -u root -p youfhe 
```

## База youfh

```
$ mysql -u root -p
CREATE DATABASE youfh DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'youfh'@'localhost' IDENTIFIED BY '<password>';
GRANT ALL PRIVILEGES ON youfh.* TO 'youfh'@'localhost';
FLUSH PRIVILEGES;
``` 

```
$ gunzip < /home/admin/tmp/youfh_2015-12-08_06h25m.Tuesday.sql.gz | mysql -u root -p youfh
```


## Код / исходник

```
$ mkdir /home/admin/tmp/puplic_html
$ mv /home/admin/web/youfhe.ru/puplic_html/dotplant2/* /home/admin/tmp/puplic_html/
$ tar xfv /home/admin/tmp/youfhe_07_12.tar.gz /home/admin/web/youfhe.ru/puplic_html/
```

## Config

* `home/igor/workspace/youfhe/config/db-local.php`



