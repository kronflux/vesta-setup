## Install PHP7 FPM

1. Remove PHP5
  This will remove PHPMYADMIN. We will reinstall it afterward.
```
  aptitude search php5|awk {'print $2'}|grep -v i386|grep -v "^A"|tr "\n"  " "
  apt-get purge <list of packages found> -y
  rm -rf /etc/php5;
```
2. Install Python things
```
apt-get install python-software-properties
```
3. Add the .deb package repo to sources
```
echo "deb http://packages.dotdeb.org jessie all" >> /etc/apt/sources.list.d/dotdeb.list
echo "deb-src http://packages.dotdeb.org jessie all" >> /etc/apt/sources.list.d/dotdeb.list
wget http://www.dotdeb.org/dotdeb.gpg -O- | apt-key add -
```
4. Update the package list
```
apt-get update
```
5. Install PHP7
```
apt-get install -y php7.0 php7.0-xsl php7.0-mbstring php7.0-zip php7.0-bcmath php7.0-common php7.0-cgi php7.0-cli php7.0-phpdbg php7.0-fpm libphp7.0-embed php7.0-dev php7.0-dbg php7.0-curl php7.0-gd php7.0-imap php7.0-interbase php7.0-intl php7.0-ldap php7.0-mcrypt php7.0-readline php7.0-odbc php7.0-pgsql php7.0-pspell php7.0-recode php7.0-tidy php7.0-xmlrpc php7.0 php7.0-json php-all-dev php7.0-sybase php7.0-sqlite3 php7.0-mysql php7.0-opcache php7.0-bz2 phpmyadmin
```
6. Remove the existing php bin dir and symlink the new one.
```
rm /usr/bin/php
ln -s /usr/bin/php7.0 /usr/bin/php
```
7. Restart the PHP7 service.
```
service php7.0-fpm start
```
8. Check you now get PHP7 on the CLI
```
php -v
```
  Hopefully you get something like:
```
      PHP 7.0.13-1~dotdeb+8.1 (cli) ( NTS )  
      Copyright (c) 1997-2016 The PHP Group  
      Zend Engine v3.0.0, Copyright (c) 1998-2016 Zend Technologies  
      with Zend OPcache v7.0.13-1~dotdeb+8.1, Copyright (c) 1999-2016, by Zend Technologies
```

## Add / Copy Vesta Templates for PHP7

1. Go to the vesta templates directory
```
cd /usr/local/vesta/data/templates/web
```
2. Copy the PHP5 directory as PHP7
```
cp -r php5-fpm/ php7.0-fpm/
```
3. Go to the Vesta Nginx templates directory
```
cd /usr/local/vesta/data/templates/web/nginx
```
4. Copt to PHP5 directory as PHP7
```
cp -r php5-fpm/ php7.0-fpm/
```
  Note: Maybe rename the old PHP5 ones instead.

5. Update Vesta Web Backend reference
```
nano /usr/local/vesta/conf/vesta.conf
```

  Update WEBBACKEND variable to php7.0-fpm

6. Update PHP socket value in new templates
```
nano /usr/local/vesta/data/templates/web/php7.0-fpm/socket.tpl
```
  Update:

  ``listen = /var/run/php5-%backend%.sock`` to ``listen = /var/run/php/php7.0-%backend%.sock``

7. Copy Vesta Web GUI template to edit PHP settings (WIP)
```
cd /usr/local/vesta/web/edit/server
cp -r php5-fpm/ php7.0-fpm
```

8. Create a new domain via the VestaCP Web GUI.

  Web Template: default
  Backend Template: default

9. Go to the public_html folder for the new domain.

  Remove the index.html and robots.txt files.

  Create an index.php containing <?php phpinfo(); ?>

  Open the domain in your browser.

  You should see the domain is using PHP7.
