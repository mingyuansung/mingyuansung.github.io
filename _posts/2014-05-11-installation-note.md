---
layout: post
title: Installation Guide
---  

# NOTE: INCOMPELTE

##Please follow the steps to build your development environment

##0. Useful Utilities
* install [macport](https://www.macports.org/install.php)
* install homebrew `ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
* optional install `brew install macvim`, this editor can open very large file fine. make sure read the prompt at the end of install to link this very app to your applicaiotn directory
* optional install [iterm](https://iterm2.com/downloads.html)
* Check with Nathan if you need `MS Office OSX` version
* Check with Nathan if you need company `Carbonite` back up support
* Check with Nathan if you ahve any IT need
* Check with Missy to grand you access to Enployee Policy, compnay directory, forms etc.  We use `Google Drive`

##1. GIT
* create your own github account and get Chris to give you permission to access company git repository
* go to https://github.com/Source-Intelligence
* fork EDEN and others to your own github account repository
* download and install GitHub for OSX
* then from https://github.com/Source-Intelligence/EDEN , clone to your desktop
* cd to your /EDEN directory
* `git remote -v` to take a look the repository you have there. You should have `origin` now
* `git remote rename origin si`, or you can use other name than si for easier typing later
* `git remote add ming https://github.com/mingyuansung/EDEN.git` to add your own REPO.  I am using my name, you can use yours or whatever name.
* cd to your /EDEN/sf2/app/config directory
* copy `parameters.dist.yml` to `parameters.yml` and update the file as needed later
* in `/etc/hosts' file, create a localhost entry such as www.sourceintelligence.local and add others as:

    ```
    127.0.0.1	localhost www.sourceintelligence.local
    255.255.255.255	broadcasthost
    ::1             localhost
    fe80::1%lo0	localhost

    # Source44 Rackspace
    50.57.94.139    s44-demobox-1
    50.57.96.170    s44-buildbox-1
    50.57.97.248    s44-stagebox-1
    192.237.167.95  s44-devbox-1
    192.237.249.70  s44-testbox-1
    198.101.155.84  s44-prodbox-1
    198.61.168.128  s44-prodbox-2
    198.61.168.126  s44-prodbox-3
    198.61.172.68   s44-prodbox-4
    23.253.37.199   s44-prodbox-5
    
    # 198.61.172.49   s44-prodbox-6
    23.253.33.132   s44-prodbox-7
    192.237.165.15  s44-prodbox-8
    162.209.94.21   s44-prodbox-10
    162.209.54.243  s44-loadbox-1
    67.207.157.99   s44-loadbox-2
    67.207.155.132  s44-combox-1
    23.253.205.245  s44-loadbox-3
    ```
* cd to your /EDEN/sf2/app directory
* `sudo mkdir cache`
* go to the newly creaed cache directory, `sudo mkdir dev`
* cd back to your EDEN/sf2 directory
* `sudo chmod +a "www allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs`
* `sudo chmod +a "www-data allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs`
* `sudo chmod +a "`whoami` allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs`

##2. XCODE
* go to install xcode form Apple App Store
* agree xcode license by running `sudo xcodebuild -license`
* run `xcode-select --install`


##3. Apache2  

* As of 2014-01 we are using 2.2.x in production  
* To install using MacPorts:  
    * `sudo port install apache2`  
    * install apache php module by `sudo port install php54-apache2handler`
    * `cd /opt/local/apache2/modules`
    * `sudo /opt/local/apache2/bin/apxs -a -e -n php5 mod_php54.so`
    * this above command should modify your httpd.conf to add in mod_php54.so module
    * if you are upgrading from previous php, make sure modify httpd.conf to remove previous php5.so link
    * to load apache on startup: `sudo port load apache2`  
* To start/stop/restart you can use apachectl  
    * symlink `apachectl` to the PATH  
    * with a MacPorts installation: `sudo ln -s /opt/local/apache2/bin/apachectl /opt/local/bin/apachectl`  
* MacPorts default config directory: `/opt/local/apache2/conf`  
* MacPorts default logs directory: `/opt/local/apache2/logs`  
* In `/etc/hosts`
    * create a localhost entry such as `www.sourceintelligence.local`
* Create an apache2 virtual host
    * I am not sure if you need to modify httpd.conf to change your ServerName and DocumentRoot.  I did.     
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
    
* Grant apache write permissions which we already did previously when we install GIT on app/logs and app/cache, see [tutorial](http://symfony.com/doc/2.1/book/installation.html#configuration-and-setup)

##4. PHP 5.4.x  

* You can use 5.4.x to run/write code compatible with the production environment (PROD still use 5.3) 
* To install using MacPorts:  
    * `sudo port install php54 +apache2 +pear`
    * in `/opt/local/apache2/conf/httpd.conf`:
        * at the end of the file add `Include conf/extra/mod_php.conf`
        * this is to let apache know how to process php file. Here is the content:
        
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
        * `brew install mongodb`
        * after installation if `php --ri mongo` says `Extension 'mongo' not present.` load the mongo.so extension   
        * modify your `/etc/php.ini` to add in `extension=mongo.so`
        * in a MacPorts installation do:  
            * `sudo sh -c 'echo "extension=mongo.so" > /opt/local/var/db/php54/mongo.ini'`  
        * `php --ri mongo` should now print driver info
        * if need manuall install, get the source tar file from Ming and unzip to any of your local directory
        * change to that unzipped directory
        * `phpize`
        * `./configure`
        * `sudo make install`
        
    * install pear by doing:
        * `wget http://pear.php.net/go-pear.phar`
        * `sudo php -d detect_unicode=0 go-pear.phar`
        * add pear to your $PATH
    * [symfony2/ClassLoader](http://pear.symfony.com/)
        * `pear channel-discover pear.symfony.com`
        * `pear install symfony2/ClassLoader`
       
    * [symfony2/Yaml](http://pear.symfony.com/)
        * `pear install symfony2/Yaml`
    * [phpunit](http://phpunit.de/manual/3.7/en/installation.html)  
        * make sure to use v 3.7.36 for our environment
        * download 'phpunit-3.7.36.phar' from 'https://phar.phpunit.de/'
        * rename it to `phpunit.phar`
        * `chmod +x phpunit.phar`
        * `sudo mv phpunit.phar /usr/local/bin/phpunit`
    * [Composer](https://getcomposer.org/doc/00-intro.md#installation-nix)
        * `curl -sS https://getcomposer.org/installer | php`
        * `mv composer.phar /usr/local/bin/composer`
    * [Node.js](http://nodejs.org/)
        * `sudo port install nodejs`
        * configure your /EDEN/sf2/app/config/parameters.yml file for the node location on your local computer.  I have to change the "node_paths:" location from the default file.
        
            ```
            [# Assetic LESS config (defaults work on Ubuntu)
            assetic.filters.less:
                node: /usr/local/bin/node
                node_paths: [/opt/local/lib/node_modules]
            //EDEN/sf2/app/config/parameters.yml  about line 12.
            ```
        * make sure you have the executable `node` in your //usr/local/bin directory. I have to copy from somewhere else.
    * npm
        * `sudo port install npm`
    * Less
        * `sudo npm -g install less@1.3.3`
        * if you need to rebuild your css, try `touch web/assets/Source44/less/base.less1`
    * install Vendor
        * cd to your EDEN directory
        * `composer install -d sf2/`
  
* In your `php.ini` (for ini file locations use `php --ini`)  

    * set the `date.timezone` to `America/Los_Angeles`  
    * increase the `memory_limit` to maybe `512M`
    * in a MacPorts installation add the following to the bottom of the file:  
        
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
    * a good free GUI for MySQL on OSX is [Sequel Pro](http://www.sequelpro.com/)
* The default installation probably has one `root` user with no password
    * modify users/passwords as needed.
    * to prevent potential problems later on, create a user with username `s44admin` and password which will be used for EDEN connections.
    * update your `/EDEN/sf2/app/config/parameters.yml` for the db name, id and password.
    * ask about getting a current database snapshot for development
* You may want to verify `mysql`, `mysqldump`, `mysqlimport`, etc, are on the PATH
    * With a OSX .dmg install you can do: `sudo ln -s /usr/local/mysql/bin/mysql /usr/local/bin/mysql`

##6. JAVA
* [use JDK 7](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)

##7. IntelliJ

* To force running under JDK 1.7 edit /Applications/IntelliJ IDEA 13.app/Contents/Info.plist file, change JVMVersion from 1.6* to 1.7* :
* Ask Chris for company license
* Go to intellij confiuration -> plugins -> browse repository and install php, php annotations, Symfony2 Plugin, Symfony2 - Clickable Views.
* Create an empty project and the module under the project shoudl point to your `/EDEN/sf2` directory.  In the module, please exclude folders `/app/cache` and `/vendor`.
* Clone `https://github.com/Source-Intelligence/misc` then get //MISC/intelliJ/source44_cs.xml style file and copy to /Users//Library/Preferences/IntelliJIdea13/codestyles/ directory. Update the IntelliJ PHP style under code style/php/Wrapping and Braces/ and check both 'Class field/constant groups/Align fields in columns' and 'Class field/constant groups/Align constants'.



