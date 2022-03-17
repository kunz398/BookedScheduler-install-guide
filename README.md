
# BookedSchediler-install-guide
disclaimer: the BookedScheduler application is developed by effgarces (https://github.com/effgarces/BookedScheduler) this is just an installation guide.
----------------------------------------

> please note this is using ubuntu 18.04
> 
> First install php 7.4

    sudo apt -y install software-properties-common

> -add repo

    sudo add-apt-repository ppa:ondrej/php
   

>     install 7.4 php

    sudo apt -y install php7.4

> check php version

    #php -v
    PHP 7.4.28 (cli) (built: Feb 17 2022 16:06:19) ( NTS )
    Copyright (c) The PHP Group
    Zend Engine v3.4.0, Copyright (c) Zend Technologies
        with Zend OPcache v7.4.28, Copyright (c), by Zend Technologies

> next we need to install all the other components

    sudo apt-get install apache2 apache2-bin apache2-data libaio1 libapache2-mod-php7.4 libapr1 libaprutil1 libdbd-mysql-perl libdbi-perl libhtml-template-perl libmysqlclient20 libterm-readkey-perl libwrap0 ssl-cert tcpd mariadb-server php7.4 php7.4-cli php7.4-common php7.4-json php7.4-mysql php7.4-readline -y

>start the services 
    
    sudo systemctl start apache2
    sudo systemctl enable apache2
    sudo systemctl start mysql
    sudo systemctl enable mysql 



> *note if there is an error with enabling or starting the mysql service,*
> *ignore and continue with the installation*
> 
> - next go to the html directory

    cd /var/www/html

> you can either git clone or download the zip for BookedScheduler and
> extract it here. I will do a git clone

    sudo git clone https://github.com/effgarces/BookedScheduler.git

next we change the ownership of the folder

    sudo chown -R www-data:www-data /var/www/html/BookedScheduler/

> next we configure mysql

    sudo mysql_secure_installation

    Set root password? [Y/n] Y
    New password: <STRONG_PASSWORD>
    Re-enter new password: <STRONG_PASSWORD>
    Remove anonymous users? [Y/n] Y
    Disallow root login remotely? [Y/n] Y
    Remove test database and access to it? [Y/n] Y
    Reload privilege tables now? [Y/n] Y

> next we create a new user and database for the BookedScheduler
> application

> login to mysql

    sudo mysql -uroot
>create database "bookeddb" and user name "booked" and password would be "password"

    MariaDB [(none)]>create database bookeddb;
    MariaDB [(none)]>create user booked@localhost identified by 'password';
    MariaDB [(none)]>grant all privileges on bookeddb.* to booked@localhost identified by 'password';
    MariaDB [(none)]>flush privileges;
    MariaDB [(none)]>exit;

> next we need to configure the config file

    cd /var/www/html/BookedScheduler/config
>edit config file with

    sudo nano config.php
    **
     * Database configuration
     */
    $conf['settings']['database']['type'] = 'mysql';
    $conf['settings']['database']['user'] = 'booked';        // database user with permission to the librebooking database
    $conf['settings']['database']['password'] = 'password';
    $conf['settings']['database']['hostspec'] = '127.0.0.1';        // ip, dns or named pipe
    $conf['settings']['database']['name'] = 'bookeddb';
>next disable captcha in the same config file

    $conf['settings']['registration.captcha.enabled'] = 'false';
>also set a installation password 

    /**
     * Installation settings
     */
    $conf['settings']['install.password'] = 'password';

>save and exit nano

>next we need to import data and schema to the current database "bookeddb"

    cd /var/www/html/BookedScheduler
>import schema

    mysql -u booked -p bookeddb < database_schema/create-schema.sql

>next import data
you would need to edit the "create-data.sql" script, there is an error in the script on line 7

>open editor to edit "create-data.sql" 

    sudo nano database_schema/create-data.sql
>change 

    insert into `layouts` values (1, 'America/New_York', 0);
>to

    insert into `layouts` values (1, 'America/New_York');

> save and exit the editor and import

  

     mysql -u booked -p bookeddb < database_schema/create-data.sql
    
>next restart apache2

    sudo systemctl restart apache2
>next go to link from browser
http://your-ip/BookedScheduler/Web/install/

>enter the installation password which is "password" in this case as set in the config file above 
next enter the database information username=booked , password=password , database name=bookeddb

>follow the instruction in the portal and installation should be successful 
