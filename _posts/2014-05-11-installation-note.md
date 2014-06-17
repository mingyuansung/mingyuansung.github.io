---
layout: post
title: Making Git Friendlier
---  

##1. Apache2  

* As of 2014-01 we are using 2.2.x in production  
* To install using MacPorts:  
    * `sudo port install apache2`  
    * To load apache on startup: `sudo port load apache2`  
* To start/stop/restart you can use apachectl  
    * Symlink `apachectl` to the PATH  
        * With a MacPorts installation: `sudo ln -s /opt/local/apache2/bin/apachectl /opt/local/bin/apachectl`  
* MacPorts default config directory: `/opt/local/apache2/conf`  
* MacPorts default logs directory: `/opt/local/apache2/logs`  
* In `/etc/hosts`
    * Create a localhost entry such as `www.sourceintelligence.local`
* Create an apache2 virtual host

    * MacPorts installations use `/opt/local/apache2/conf/extra/httpd-vhosts.conf`  
        
    ```  
    <VirtualHost *:80>
        ServerName www.sourceintelligence.local
        DocumentRoot "/fully/qualified/path/to/EDEN/sf2/web"

    <Directory "/fully/qualified/path/to/EDEN/sf2/web">
        Options -Indexes
        AllowOverride All
        Allow from All
    </Directory>
    </VirtualHost>
    ```
* Grant apache write permissions on app/logs and app/cache, see [tutorial](http://symfony.com/doc/2.1/book/installation.html#configuration-and-setup)

##2. PHP 5.3.x  

* You must use 5.3.x to run/write code compatible with the production environment  
* To install using MacPorts:  
    * `sudo port install php5 +apache2 +pear`
    * `cd /opt/local/apache2/modules`
    * `sudo /opt/local/apache2/bin/apxs -a -e -n "php5" libphp5.so`
    * In `/opt/local/apache2/conf/httpd.conf`:
        * At the end of the file add `Include conf/extra/mod_php.conf`
    * You may want to copy and symlink your php.ini
        * `sudo cp /opt/local/etc/php5/php.ini-development /opt/local/etc/php5/php.ini`
        * `sudo ln -s /opt/local/etc/php5/php.ini /etc/php.ini`
* Install PHP extensions:
    * Via MacPorts:  
    ```
sudo port install php5-apc php5-curl php5-iconv php5-intl \
    php5-mbstring php5-mcrypt php5-mysql php5-openssl php5-posix php5-xdebug  
    ```  
* The following PECL/PEAR libraries should be installed (follow links for install docs):
    * [mongo](http://docs.mongodb.org/ecosystem/drivers/php/)
        * After installation if `php --ri mongo` says `Extension 'mongo' not present.` load the mongo.so extension  
        * In a MacPorts installation do:  
            * `sudo sh -c 'echo "extension=mongo.so" > /opt/local/var/db/php5/mongo.ini'`  
        * `php --ri mongo` should now print driver info
    * [symfony2/ClassLoader](http://pear.symfony.com/)
    * [symfony2/Yaml](http://pear.symfony.com/)
    * [phpunit](http://phpunit.de/manual/3.7/en/installation.html)  
  
* In your `php.ini` (for ini file locations use `php --ini`)  

    * Set the timezone to `America/Los_Angeles`  
    * In a MacPorts installation add the following to the bottom of the file:  
        
        ```
        [xdebug]
            xdebug.remote_autostart=1
            xdebug.remote_enable=1

        [apc]
            apc.shm_size=128M
```  

##3. MySQL

* As of 2014-01 we are using 5.5.x in production
* On OSX download and install the correct version 64-bit dmg from the link above
    * A good free GUI for MySQL on OSX is [Sequel Pro](http://www.sequelpro.com/)
* The default installation probably has one `root` user with no password
    * Modify users/passwords as needed.
    * To prevent potential problems later on, create a user with username `s44admin` which will be used for EDEN connections.
    * Ask about getting a current database snapshot for development
* You may want to verify `mysql`, `mysqldump`, `mysqlimport`, etc, are on the PATH
    * With a OSX .dmg install you can do: `sudo ln -s /usr/local/mysql/bin/mysql /usr/local/bin/mysql`

