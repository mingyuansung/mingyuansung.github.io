---
layout: post
title: Installation Guide
---  

##0. Useful Utilities
* install [macport](https://www.macports.org/install.php)
* install homebrew `ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
* `brew install macvim`
* install [iterm](https://iterm2.com/downloads.html)

##1. GIT
* create your own git account and get Chris to give you permission to access company git source
* go to https://github.com/Source-Intelligence
* fork EDEN and others to your own git account
* download and install GitHub for OSX
* clone to your desktop
* cd to your EDEN/sf2/app directory
* `mkdir cache`
* cd to your EDEN/sf2 directory
* `sudo chmod +a "www allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs`
* `sudo chmod +a "`whoami` allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs`

##2. XCODE
* go to install xcode form Apple App Store
* agree xcode license `sudo xcodebuild -license`
* run `xcode-select --install`


##3. Apache2  

* As of 2014-01 we are using 2.2.x in production  
* To install using MacPorts:  
    * `sudo port install apache2`  
    * install apache php module by `sudo port install php54-apache2handler`
    * `cd /opt/local/apache2/modules`
    * `sudo /opt/local/apache2/bin/apxs -a -e -n php5 mod_php54.so`
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

##4. PHP 5.3.x  

* You must use 5.4.x to run/write code compatible with the production environment  
* To install using MacPorts:  
    * `sudo port install php54 +apache2 +pear`
    * `cd /opt/local/apache2/modules`
    * `sudo /opt/local/apache2/bin/apxs -a -e -n "php5" libphp54.so`
    * In `/opt/local/apache2/conf/httpd.conf`:
        * At the end of the file add `Include conf/extra/mod_php.conf`
        * THis is to let apache know how to process php file. Here is the content:
        
        ```  
        <IfModule php5_module>
            AddType application/x-httpd-php .php
            AddType application/x-httpd-php-source .phps

            <IfModule dir_module>
                DirectoryIndex index.html index.php
            </IfModule>
        </IfModule>
        ```
    * You may want to copy and symlink your php.ini
        * `sudo cp /opt/local/etc/php5/php.ini-development /opt/local/etc/php5/php.ini`
        * `sudo ln -s /opt/local/etc/php5/php.ini /etc/php.ini`
* Install PHP extensions:
    * Via MacPorts:  
    ```
sudo port install php54-apc php54-curl php54-iconv php54-intl
    php54-mbstring php54-mcrypt php54-mysql php54-openssl php54-posix php54-xdebug  
    ```  
* The following PECL/PEAR libraries should be installed (follow links for install docs):
    * [mongo](http://docs.mongodb.org/ecosystem/drivers/php/)
        * After installation if `php --ri mongo` says `Extension 'mongo' not present.` load the mongo.so extension  
        * In a MacPorts installation do:  
            * `sudo sh -c 'echo "extension=mongo.so" > /opt/local/var/db/php5/mongo.ini'`  
        * `php --ri mongo` should now print driver info
        * if need manuall install, get the source tar file from Ming and unzip to any of your local directory
        * change to that unzipped directory
        * `phpize`
        * `./configure`
        * `sudo make install`
        
    * [symfony2/ClassLoader](http://pear.symfony.com/)
        * `pear channel-discover pear.symfony.com`
        * `pear install symfony2/ClassLoader`
       
    * [symfony2/Yaml](http://pear.symfony.com/)
        * `pear install symfony2/Yaml`
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

##5. MySQL

* As of 2014-01 we are using 5.5.x in production
* On OSX download and install the correct version 64-bit dmg from the link above
    * A good free GUI for MySQL on OSX is [Sequel Pro](http://www.sequelpro.com/)
* The default installation probably has one `root` user with no password
    * Modify users/passwords as needed.
    * To prevent potential problems later on, create a user with username `s44admin` and password `test` which will be used for EDEN connections.
    * Ask about getting a current database snapshot for development
* You may want to verify `mysql`, `mysqldump`, `mysqlimport`, etc, are on the PATH
    * With a OSX .dmg install you can do: `sudo ln -s /usr/local/mysql/bin/mysql /usr/local/bin/mysql`

##6. JAVA
* [use JDK 7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)

##7. IntelliJ

* To force running under JDK 1.7 edit /Applications/<Product>.app/Contents/Info.plist file, change JVMVersion from 1.6* to 1.7* :
* Ask Chris for company license



