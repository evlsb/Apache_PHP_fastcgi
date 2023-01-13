1) Установите файл vc_redist.x64.exe

2) Создание структуры веб-сервера

	Создадим структуру каталогов нашего сервера. Главная идея - разделить исполнимые файлы и файлы сайтов с базами данных. Это удобно для обслуживания сервера, в том числе для резервного копирования.

В корне диска C:\ создайте каталог Server. В этом каталоге создайте 2 подкаталога: bin (для исполнимых файлов) и data.
Перейдите в каталог data и там создайте подпапки DB (для баз данных) и htdocs (для сайтов).
Перейдите в каталог C:\Server\data\DB\ и создайте там пустую папку data.

3) Установка Apache 2.4

	Содержимое скаченного архива (точнее говоря, только каталог Apache24), распакуйте в C:\Server\bin\.
Перейдите в каталог c:\Server\bin\Apache24\conf\ и откройте файл httpd.conf любым текстовым редактором.
В нём нам нужно заменить ряд строк.

	Меняем:
	Define SRVROOT "c:/Apache24"
	на
	Define SRVROOT "c:/Server/bin/Apache24"

	меняем:
	#ServerName www.example.com:80
	на
	ServerName localhost

	меняем:
	Options Indexes FollowSymLinks
	на
	Options Indexes FollowSymLinks ExecCGI


	{DocumentRoot – это директория, где по умолчанию находятся сайты. По умолчанию именно там ищутся все пришедшие на веб-сервер запросы. Укажите здесь путь до корневой папки в веб-документами. Конечный слэш писать не нужно.}
	меняем:
	DocumentRoot "${SRVROOT}/htdocs"
	на
	DocumentRoot "c:/Server/data/htdocs"

	меняем:
	<Directory "${SRVROOT}/htdocs">
	на
	<Directory "c:/Server/data/htdocs">

	меняем:
	DirectoryIndex index.html
	на
	DirectoryIndex index.php index.html index.htm

	{Директива AllowOverride установлена на None, это означает запрет использовать файлы .htaccess.}
	меняем:
	# AllowOverride controls what directives may be placed in .htaccess files.
	# It can be "All", "None", or any combination of the keywords:
	#   AllowOverride FileInfo AuthConfig Limit
	#
	AllowOverride None
	на
	# AllowOverride controls what directives may be placed in .htaccess files.
	# It can be "All", "None", or any combination of the keywords:
	#   AllowOverride FileInfo AuthConfig Limit
	#
	AllowOverride All

	и меняем
	#LoadModule rewrite_module modules/mod_rewrite.so
	на
	LoadModule rewrite_module modules/mod_rewrite.so

Сохраняем и закрываем файл. Всё, настройка Apache завершена!
Откройте командную строку (это можно сделать нажав одновременно клавиши Win+X). Выберите там Windows PowerShell (администратор) и скопируйте туда:

	c:\Server\bin\Apache24\bin\httpd.exe -k install

Если поступит запрос от файервола в отношение Apache, то нажмите Разрешить.

Теперь вводим в командную строку:

	c:\Server\bin\Apache24\bin\httpd.exe -k start

И нажмите Enter.
Теперь в браузере набираем http://localhost/



5) Настройка виртуальных хостов

Из архива "mod_fcgid-2.3.10-win64-VS17" копируем модуль "mod_fcgid.so" в папку "C:\Server\bin\Apache24\modules"
В конец файла c:\Server\bin\Apache24\conf\httpd.conf добавляем:
	LoadModule fcgid_module modules/mod_fcgid.so

В папке c:\Server\bin\ создаём каталог PHP и копируем в него содержимое архива php-8.0.0-Win32-vs16-x64.zip.

В файле c:\Server\bin\Apache24\conf\httpd.conf:
	меняем
	# Virtual hosts
	#Include conf/extra/httpd-vhosts.conf
	на
	# Virtual hosts
	Include conf/extra/httpd-vhosts.conf

	добавляем
	# FastCGI
	Include conf/extra/httpd-fastcgi.conf

	создаем в каталоге conf/extra/ файл httpd-fastcgi.conf
	добавляем в него:
	FcgidInitialEnv PATH "c:/Server/bin/PHP;C:/WINDOWS/system32;C:/WINDOWS;C:/WINDOWS/System32/Wbem;"
	FcgidInitialEnv SystemRoot "C:/Windows"
	FcgidInitialEnv SystemDrive "C:"
	FcgidInitialEnv TEMP "C:/WINDOWS/Temp"
	FcgidInitialEnv TMP "C:/WINDOWS/Temp"
	FcgidInitialEnv windir "C:/WINDOWS"
	FcgidIOTimeout 64
	FcgidConnectTimeout 16
	FcgidMaxRequestsPerProcess 1000 
	FcgidMaxProcesses 50 
	FcgidMaxRequestLen 8131072
	# Location php.ini:
	FcgidInitialEnv PHPRC "c:/Server/bin/PHP"
	FcgidInitialEnv PHP_FCGI_MAX_REQUESTS 1000

	<Files ~ "\.php$>"
  	AddHandler fcgid-script .php
  	FcgidWrapper "c:/Server/bin/PHP/php-cgi.exe" .php
	</Files>

В файле conf/extra/httpd-vhosts.conf добавляем виртуальные хосты:
	<VirtualHost 127.0.0.1:80>
    		ServerName site1.com
    		ServerAdmin webmaster@dummy-host.example.com
    		DocumentRoot "c:/Server/data/htdocs/site1"
	</VirtualHost>

	<VirtualHost 127.0.0.1:80>
    		ServerName site2.com
    		ServerAdmin webmaster@dummy-host.example.com
    		DocumentRoot "c:/Server/data/htdocs/site2"
	</VirtualHost>

В файле C:\Windows\System32\drivers\etc\hosts добавляем строчки:
	127.0.0.1       site1.com
	127.0.0.1       site2.com

В каталогах C:\Server\data\htdocs создаём папки site1, site2 с файлами index.php, добавляем:
	<?php phpinfo (); ?>

И перезапускаем Apache

	c:\Server\bin\Apache24\bin\httpd.exe -k restart


В браузере откройте ссылку site1.com и site1.com

4) Установка и настройка MySQL 8.0

В каталог bin распаковываем файлы MySQL (из архива mysql-8.0.11-winx64.zip). Переименовываем папку mysql-8.0.11-winx64 в mysql-8.0 (для краткости). Кстати, распакованная папка mysql-8.0 занимает около гигабайта!
Заходим в эту папку и создаём там файл my.ini Теперь открываем этот файл любым текстовым редактором.
Добавьте туда следующие строки:

	[mysqld]
 
	sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
	datadir="c:/Server/data/DB/data/"
	default_authentication_plugin=mysql_native_password

Сохраните и закройте его.
Настройка завершена, но нужно ещё выполнить инициализацию и установку, для этого открываем командную строку от имени администратора и последовательно вводим туда:

	C:\Server\bin\mysql-8.0\bin\mysqld --initialize-insecure --user=root
	C:\Server\bin\mysql-8.0\bin\mysqld --install
	net start mysql

По окончанию этого процесса в каталоге C:\Server\data\DB\data\ должны появиться автоматически сгенерированные файлы.
Теперь служба MySQL будет запускаться при каждом запуске Windows.

 

5) Установка и настройка PHP 8

Настройка PHP происходит в файле php.ini. В zip-архивах, предназначенных для ручной установки и для обновлений, php.ini нет (это сделано специально, чтобы случайно не затереть ваш файл, с вашими уникальными настройками). Зато есть два других, которые называются php.ini-development и php.ini-production. Любой из них, при ручной установке, можно переименовать в php.ini и настраивать дальше. На локалхосте мы будет использовать php.ini-development.

Открываем файл php.ini любым текстовым редактором, ищем строчку

	меняем:
	;extension_dir = "ext"
	на
	extension_dir = "C:\Server\bin\PHP\ext\"

	меняем:
	;extension=bz2
	;extension=curl
	;extension=ffi
	;extension=ftp
	;extension=fileinfo
	;extension=gd
	;extension=gettext
	;extension=gmp
	;extension=intl
	;extension=imap
	;extension=ldap
	;extension=mbstring
	;extension=exif      ; Must be after mbstring as it depends on it
	;extension=mysqli
	;extension=oci8_12c  ; Use with Oracle Database 12c Instant Client
	;extension=odbc
	;extension=openssl
	;extension=pdo_firebird
	;extension=pdo_mysql
	;extension=pdo_oci
	;extension=pdo_odbc
	;extension=pdo_pgsql
	;extension=pdo_sqlite
	;extension=pgsql
	;extension=shmop
	на
	extension=bz2
	extension=curl
	extension=ffi
	extension=ftp
	extension=fileinfo
	extension=gd
	extension=gettext
	extension=gmp
	extension=intl
	extension=imap
	extension=ldap
	extension=mbstring
	extension=exif      ; Must be after mbstring as it depends on it
	extension=mysqli
	;extension=oci8_12c  ; Use with Oracle Database 12c Instant Client
	extension=odbc
	extension=openssl
	;extension=pdo_firebird
	extension=pdo_mysql
	;extension=pdo_oci
	extension=pdo_odbc
	extension=pdo_pgsql
	extension=pdo_sqlite
	extension=pgsql
	extension=shmop

	меняем:
	;extension=soap
	;extension=sockets
	;extension=sodium
	;extension=sqlite3
	;extension=tidy
	;extension=xsl
	на
	extension=soap
	extension=sockets
	extension=sodium
	extension=sqlite3
	extension=tidy
	extension=xsl

Из архива SQLSRV510 копируем библиотеки php_pdo_sqlsrv_81_nts_x64.dll и php_sqlsrv_81_nts_x64.dll в папку с расширениями C:\Server\bin\PHP\ext

Открываем файл php.ini и добавляем расширения:
	extension=php_pdo_sqlsrv_81_nts_x64.dll
	extension=php_sqlsrv_81_nts_x64.dll

Устанавливаем драйвер ODBC 17 из файлов: msodbcsql.msi


Этими действиями мы включили расширения. Они могут понадобиться в разных ситуациях для разных скриптов. Сохраняем файл и перезапускаем Apache.

6) Установка и настройка phpMyAdmin

В каталог c:\Server\data\htdocs\ копируем содержимое архива phpMyAdmin-4.5.1-all-languages.zip. Переименовываем phpMyAdmin-4.5.1-all-languages в phpmyadmin (для лаконичности)

В каталоге c:\Server\data\htdocs\phpmyadmin\ создаём файл config.inc.php и копируем туда:

	<?php
 
	/* Servers configuration */
	$i = 0;
 
	/* Server: localhost [1] */
	$i++;
	$cfg['Servers'][$i]['verbose'] = '';
	$cfg['Servers'][$i]['host'] = 'localhost';
	$cfg['Servers'][$i]['port'] = '';
	$cfg['Servers'][$i]['socket'] = '';
	$cfg['Servers'][$i]['connect_type'] = 'tcp';
	$cfg['Servers'][$i]['auth_type'] = 'cookie';
	$cfg['Servers'][$i]['user'] = 'root';
	$cfg['Servers'][$i]['password'] = '';
	$cfg['Servers'][$i]['nopassword'] = true;
	$cfg['Servers'][$i]['AllowNoPassword'] = true;
 
	/* End of servers configuration */
 
	$cfg['blowfish_secret'] = 'kjLGJ8g;Hj3mlHy+Gd~FE3mN{gIATs^1lX+T=KVYv{ubK*U0V';
	$cfg['DefaultLang'] = 'ru';
	$cfg['ServerDefault'] = 1;
	$cfg['UploadDir'] = '';
	$cfg['SaveDir'] = '';
 
	?>

В браузере набираем http://localhost/phpmyadmin/
В качестве имя пользователя вводим root. Поле пароля оставляем пустым. 

7) Использование сервера и бэкап данных

В каталоге c:\Server\data\htdocs\ создавайте папки и файлы, например:
c:\Server\data\htdocs\test\ajax.php – этот файл, соответственно, будет доступен по адресу http://localhost/test/ajax.php и т.д.

Для создания полного бэкапа всех сайтов и баз данных достаточно скопировать каталог C:\Server\data\.
Перед обновлением модулей, делайте бэкап папки bin – в случае возникновения проблем, можно будет легко откатиться к предыдущим версиям.
При повторной установке сервера или при его обновлении, необходимо заново настраивать конфигурационные файлы. Если у вас есть копии этих файлов, то процесс можно значительно ускорить. Желательно забэкапить следующие файлы:

	c:\Server\bin\Apache24\conf\httpd.conf
	c:\Server\bin\mysql-8.0\my.ini
	c:\Server\bin\PHP\php.ini
	c:\Server\data\htdocs\phpMyAdmin\config.inc.php

В них и хранятся все настройки.

8) Дополнительная настройка PHP

PHP в настоящее время очень мощный, гибкий, удобный инструмент. На локальном компьютере с помощью него можно решать разнообразные задачи, совсем не обязательно связанные с генерацией Web-страниц. При решении неординарных задач можно упереться в ограничения, установленные в настройках. Эти настройки содержаться в файле php.ini (c:\Server\bin\PHP\php.ini) Рассмотрим некоторые из них:

	memory_limit = 128M - устанавливает максимальное количество памяти, которое может использовать скрипт

	post_max_size = 8M - устанавливает максимальное количество данных, которые будут приняты при отправке методом POST

	;default_charset = "UTF-8" - устанавливает кодировку (по умолчанию, строка закомментирована)

	upload_max_filesize = 2M - максимальный размер загружаемого на сервер файла. Изначально установлен очень маленький размер – только два мегабайта. Например, при загрузке базы данных в phpMyAdmin, не получится загрузить файл больше 2 мегабайт до тех пор, пока не будет изменён этот пункт настройки.

	max_file_uploads = 20 - максимальное количество файлов для загрузки за один раз

	max_execution_time = 30 - максимальное время выполнения одного скрипта

Менять эти настройки совершенно необязательно, но полезно о них знать.

9) Удаление сервера

Если сервер вам больше не нужен, либо вы хотите установить его заново, остановите службы и удалите их из автозапуска последовательно выполнив в командной строке:

	c:\Server\bin\Apache24\bin\httpd.exe -k stop
	c:\Server\bin\Apache24\bin\httpd.exe -k uninstall
	net stop mysql
	c:\Server\bin\mysql-8.0\bin\mysqld --remove

Удалите файлы сервера, для этого удалите папку C:\Server\. Внимание, это удалит все базы данных и ваши сайты.