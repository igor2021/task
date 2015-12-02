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

# 1.1. Установка nginx

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

Если мы собираемся использовать только сервер `nginx`, то отключим службу `apache2`:

```
# service apache2 stop
  * Stopping web server apache2
# service apache2 status
  * apache2 is not running
```

Так же отключим автозагрузку службы `apache2`:

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

Чтобы обратно влкючить автозагрузку службы `apache2` необходимо выполнить:

```
# sysv-rc-conf apache2 on
```

PS. Также можно посмотреть какие службы установлены по ссылке: https://188.166.17.183:8083/list/server/

# 1.2. Настройка Nginx

* Заметка
  * Основные настройки `nginx` хранятся в файле `/etc/nginx/nginx.conf`.
  * Настройки сайта (по ip: 188.166.17.183), храняться в файле `/etc/nginx/conf.d/188.166.17.183.conf`.
  
Сделаем копию файла `/etc/nginx/conf.d/188.166.17.183.conf`:


```
# cp /etc/nginx/conf.d/188.166.17.183.conf /etc/nginx/conf.d/188.166.17.183.conf.
```

Но возникает вопрос? Где же прописан порт `8083`. Попробуем это найти:

```
# grep -rl '8083' /etc/nginx
```

Результат поиска `null`. 




  

