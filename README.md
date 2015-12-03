Hosting
====================

# Task

Текст задания как есть (но далее будем корректировать):

```
1. ОС:Ubuntu  
2. Админ панель? возможность работать и смотреть графики 
3. Установить пакеты Dotplant2 YII2
http://docs.dotplant.ru/ru/setup-example.html установить на домен youfhe.ru 
установить DotPlant2
4. Установить пакеты.

nginx
node npm
redis 
redis-desktop-manager http://redisdesktop.com/download
ssh доступ по ip http://rubuntu.com/1029/%D1%81%D0%B4%D0%B5%D0%BB%D0%B0%D1%82%D1%8C-%D0%B4%D0%BE%D1%81%D1%82%D1%83%D0%BF-%D0%BE%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%BD%D0%BE%D0%B3%D0%BE
добавить 87.117.167.118
#rabitmq
git
composer

postgerSQL

5. Настройка
почтовые почтовые уведомления

настройка backup mysql http://sourceforge.net/projects/automysqlbackup/
настройка backup сайта крон больше 3 дней


gunzip youfh_2015-03-10_06h47m.Tuesday.sql.gz
mysql -u root -p youfh < youfh_2015-04-08_11h53m.Wednesday.sql
tar -cvzf mylinker_02_07.tar.gz /var/www/mylinker
tar -xvf youfh_19_05_afternoon.tar.gz -C /var/www/archive/
```


# Порядок выполнения задания

# 1.1. Проверка установки nginx

Есть подозрение что `nginx` уже установлен или установлен какой-то и из веб-серверов (т.к. имеем возможность открывать ссылку: https://188.166.17.183:8083). Проверим это:

```
# service nginx status
  * nginx is running
```

Т.е. видим что сервер `nginx` существует и запущен.

```
# service apache2 status
  * apache2 is running
```

Т.е. сервер `apache2` тоже существует и запущен.

Если мы собираемся использовать только сервер `nginx`, тогда отключим службу `apache2`:

```
# service apache2 stop
  * Stopping web server apache2
# service apache2 status
  * apache2 is not running
```

Так же можем отключим автозагрузку службы `apache2`:

```
# apt-get install sysv-rc-conf
```

```
# sysv-rc-conf apache2 off
```

Проверить режимы автозагрузки можно командой:

```
# sysv-rc-conf --list
```

Чтобы обратно влкючить автозагрузку службы `apache2` необходимо выполнить (включим т.к. точно не знаем будем ли полностью отказыватся от службы):

```
# sysv-rc-conf apache2 on
```

PS. Также можно посмотреть какие службы установлены по ссылке: https://188.166.17.183:8083/list/server/

# 1.2. Обзор Nginx

* Заметка
  * Основные настройки `nginx` хранятся в файле `/etc/nginx/nginx.conf`.
  * Настройки сайта (по ip: 188.166.17.183), храняться в файле `/etc/nginx/conf.d/188.166.17.183.conf`.
  * Настройки для "Vesta Control panel", хранятся в файле `/etc/nginx/conf.d/vesta.conf`.
  
Но возникает вопрос? Где же прописан порт `8083`. Попробуем это найти:

```
# grep -rl '8083' /etc/nginx
```

Результат поиска `null`. 

Есть подозрение что тут замешан iptables/firewall. Попробуем это проверить:

```
# iptables -L
```

```
# cat /etc/iptables.rules
```

Т.е. действительно видем записи:

```
-A INPUT -p tcp -m tcp --dport 8083 -j ACCEPT
-A INPUT -p tcp -m tcp --sport 8083 -j ACCEPT
```

Но тут нет перенаправления. Откуда же взялся порт `8083`?
Заходим на сайт vestcap.com по сслыке `http://vestacp.com/docs/`. Видим что этот порт занять и имеет специальне назначение для "Vesta Control panel".

Тажке упоминание об этом порте `8083` есть в файле `/etc/roundcube/plugins/password/config.inc.php`.



# 1.3. Проверка и установка PostgerSQL

Проверим статус службы `postgresql`:
```
# service postgresql status
postgresql: unrecognized service
```

Проверим установлен ли пакет `postgresql`:

```
# dpkg -s postgresql | grep Status
```

Предварительно также читаем: `http://vestacp.com/docs/` (How to set up PostgreSQL on a Debian or Ubuntu).

Установим пакет `postgresql`:

```
# apt-get install postgresql postgresql-contrib phppgadmin
Setting up phppgadmin (5.1-1) ...
 * Reloading web server apache2         * 
 * Apache2 is not running
```

Видим, что `phppgadmin` спрашивает `Apache2`. Поэтому включим эту службу (см. п. 1.1).

```
# cp /etc/postgresql/*/main/pg_hba.conf /etc/postgresql/*/main/pg_hba.conf.src
# wget http://c.vestacp.com/0.9.8/debian/pg_hba.conf -O /etc/postgresql/*/main/pg_hba.conf
```

```
# service postgresql restart
```

```
# su - postgres
psql -c "ALTER USER postgres WITH PASSWORD '<password>'"
exit
```

```
# vi /usr/local/vesta/conf/vesta.conf
...
DB_SYSTEM='mysql,pgsql'
```

```
# v-add-database-host pgsql localhost postgres <password>
```

```
# cp /etc/phppgadmin/config.inc.php /etc/phppgadmin/config.inc.php.src
# wget http://c.vestacp.com/0.9.8/debian/pga.conf -O /etc/phppgadmin/config.inc.php
# mkdir /etc/apache2/conf.d.src
# cp /etc/apache2/conf.d/phppgadmin /etc/apache2/conf.d.src/phppgadmin
# wget http://c.vestacp.com/0.9.8/debian/apache2-pga.conf -O /etc/apache2/conf.d/phppgadmin
```

```
# service apache2 restart
```



# 2. Создание пользователя

Создадим нового пользователя в каталоге которого разместим CMS "Dotplant2".


```
# adduser ecom
```

```
# passwd ecom
	password: <password>
```

Но необходимо также учесть чтобы пользователи отображались в "Vesta Control panel". 
Плоэтому читаем http://vestacp.com/docs/ (How to set up PostgreSQL on a Debian or Ubuntu).Или добавляем пользователей через саму панель.

# 3.1. Установка необходимых пакетов для CMS "Dotplant2"  



# 3.2. Установка CMS "Dotplant2"

Вся установка будет производится от пользователя `ecom`.

 
