# php7-from-scratch
Instructions on how to compile PHP 7.1 from source on Ubuntu 16.04 machines. This
will also include instructions on how to configure the machine for Nginx.

These instructions are ideal for those who want to run PHP in production.

## Install packages

    sudo apt-get install \
    build-essential libssl-dev libcurl4-openssl-dev \
    zlib1g-dev curl libxml2-dev libreadline-dev systemd \
    bison dkms kpartx openssl pkg-config
    
    # install MySQL
    sudo apt-get install mysql-server mysql-client
    
    # install postgres 9.5
    sudo apt-get install postgresql postgresql-contrib postgresql-server-dev-9.5
    
    # install nginx
    apt-get install nginx

## Download Latest PHP 7.1.1
As of the time of this, PHP 7.1.1 was the latest PHP available. Download the
tarball (I'll assume BZip) and extract it.

__NOTE: this installs PHP in the user's local bin. In this example, the user is
named me. Change this__

    cd /tmp;
    wget http://php.net/get/php-7.1.1.tar.bz2/from/this/mirror -O php-7.1.1.tar.bz2
    tar xf php-7.1.1.tar.bz2
    cd php-7.1.1
    mkdir -p /home/me/bin/php7
    cat >> build_php.sh
    #!/bin/sh

    ./configure --prefix=/home/me/bin/php7 \
        --enable-bcmath \
        --enable-fpm \
        --enable-ftp \
        --enable-mbstring \
        --enable-mysqlnd \
        --enable-shmop \
        --enable-sockets \
        --enable-sysvmsg \
        --enable-sysvsem \
        --enable-sysvshm \
        --enable-zip \
        --with-curl \
        --with-mysqli \
        --with-mysql-sock=/var/run/mysqlnd \
        --with-pear \
        --with-openssl \
        --with-pdo-mysql \
        --with-readline \
        --with-zlib \
        --enable-pcntl \
        --with-readline \
        --with-pgsql

    # press ctrl+c to exit out of cat

#### compile and install
Remember, PHP will be installed in the home directory.

    sh build_php.sh
    make
    make install

#### Edit the PHP.ini file
Copy the *php-7.1.1/php.ini-development* file from the source directory to the *~/bin/php7/lib/* directory, and rename
the file to *php.ini*

    cp /tmp/php-7.1.1/php.ini-development ~/bin/php7/lib/php.ini

You'll need to change a few settings

Change your timezone

    [Date]
    ; Defines the default timezone used by the date functions
    ; http://php.net/date.timezone
    date.timezone = America/Chicago
    
The MySQL socket PDO connects to

    ; Default socket name for local MySQL connects.  If empty, uses the built-in
    ; MySQL defaults.
    ; http://php.net/pdo_mysql.default-socket
    pdo_mysql.default_socket=/var/run/mysqld/mysqld.sock

PHP-FPM fix (You will most likely have to add this to the file)

    [php-fpm]
    cgi.fix_pathinfo=0:

You need to create a *php-fpm.conf* file to use Nginx.

    cd /home/me/bin/php7/etc/; cp php-fpm.conf.default php-fpm.conf
    cd /home/me/bin/php7/etc/php-fpm.d/; cp www.conf.default www.conf

In your favorite editor, open the www.conf file, and change the user and group
from nobody to www-data (the user for nginx), and change the listen port to use
sockets

    [editor] www.conf # this is the file for Nginx
    ...
    user = www-data
    group = www-data
    ...
    listen = /var/run/php-fpm.sock

    ...
    ; Set permissions for unix socket, if one is used. In Linux, read/write
    ; permissions must be set in order to allow connections from a web server. Many
    ; BSD-derived systems allow connections regardless of permissions.
    ; Default Values: user and group are set as the running user
    ;                 mode is set to 0660
    listen.owner = www-data
    listen.group = www-data
    listen.mode = 0660

Assuming you'll be logged into your server not as root, add the PHP binary to
your .bashrc profile

    # open ~/.bashrc in your favorite editor and add this to the bottom
    if [ -d "$HOME/bin" ] ; then
        PATH="$HOME/bin:$PATH"
    fi


Now, add PHP 7 binary to your path by:

    cd /home/me/bin
    ln -s php7/bin/php php
    ln -s php7/bin/php-cgi php-cgi
    ln -s php7/bin/php-config php-config
    ln -s php7/bin/phpize phpize
    ln -s php7/bin/phar.phar phar
    ln -s php7/bin/pear pear
    ln -s php7/bin/phpdbg phpdbg
    ln -s php7/sbin/php-fpm php-fpm
    
### Start PHP-FPM
You'll have to start *php-fpm* manually to get it working with NginX

    sudo ~/bin/php7/sbin/php-fpm

## WEBSERVERS

__Laravel 5.2__
If you're using Laravel 5.2 this is how you start the built in PHP server

    php artisan serve --host=your.ip.of.vm --port=8000

__Symfony 2__
If you're using Symfony 2, this is how you start the built in PHP server:

    php bin/console server:start {your.ip.of.vm}:{port}

And how you stop

    php bin/console server:stop  {your.ip.of.vm}:{port}

__Nginx__ If you're using nginx: install Ubuntu's version

    sudo apt-get install nginx

You can modify the default config, but I'm going to make copy and use the copy

    cd /etc/nginx/sites-available; cp default myphp7

Open *myphp7* and change the root directory to where your code is. Since, I'm using Parallels, my directory would look like this

    root /media/psf/my_laravel5.2_project;

 Add index.php to the index list

    index indext.html index.htm index.php

In the *location* section, add `/index.php?query_string` to the try_files section

    try_files $uri $uri/ /index.php?query_string;

In the FastCGI section, make the appropriate changes

    location ~ \.php$ {
            # use for 0 day exploits
            try_files $uri /index.php =404;
            
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            # With php-fpm:
            fastcgi_pass unix:/var/run/php-fpm.sock;
            fastcgi_index index.php;
            include fastcgi_params;
    }

Change to the sites-enabled directory, remove the default site, and make a link to the the new PHP config you just created

    cd /etc/nginx/sites-enabled
    sudo unlink default
    sudo ln -s /etc/nginx/sites-available/myphp7 myphp7
    sudo nginx -s reload

Nginx should now be serving your files

#### Nginx Virtualbox Bug
You'll need to edit `/etc/nginx/nginx.conf` and change the line `sendfile on;` to `sendfile off;`

What will happen is changes to static files (CSS and Javascript) files won't be updated. Turning off _sendfile_ will cause Nginx to serve the file via a different method and the new file's changes will be displayed immediately

