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

## Download PHP 7.1.0
As of the time of this, PHP 7.1.0 was the latest PHP available. Download the
tarball (I'll assume BZip) and extract it.

__NOTE: this installs PHP in the user's local bin. In this example, the user is
named me. Change this__

    cd /tmp;
    wget http://us3.php.net/get/php-7.1.0.tar.bz2/from/this/mirror -O php-7.1.0.tar.bz2
    tar xjf php-7.1.0.tar.bz2
    cd php-7.1.0/
    mkdir -p /home/me/bin/php7.1
    cat >> build_php7.1.sh
    #!/bin/sh

    ./configure --prefix=/home/me/bin/php7.1 \
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

    sh build_php7.1.sh
    make
    make install

#### You're PHP.ini

    cat >> ~/bin/php7/lib/php.ini

    [Date]
    ; Defines the default timezone used by the date functions
    ; http://php.net/date.timezone
    date.timezone = America/Chicago

    [Pdo_mysql]
    ; If mysqlnd is used: Number of cache slots for the internal result set cache
    ; http://php.net/pdo_mysql.cache_size
    pdo_mysql.cache_size = 2000

    ; Default socket name for local MySQL connects.  If empty, uses the built-in
    ; MySQL defaults.
    ; http://php.net/pdo_mysql.default-socket
    pdo_mysql.default_socket=/var/run/mysqld/mysqld.sock

    [MySQL]
    ; Allow accessing, from PHP's perspective, local files with LOAD DATA statements
    ; http://php.net/mysql.allow_local_infile
    mysql.allow_local_infile = On

    ; Allow or prevent persistent links.
    ; http://php.net/mysql.allow-persistent
    mysql.allow_persistent = On

    ; If mysqlnd is used: Number of cache slots for the internal result set cache
    ; http://php.net/mysql.cache_size
    mysql.cache_size = 2000
    ; Maximum number of persistent links.  -1 means no limit.
    ; http://php.net/mysql.max-persistent
    mysql.max_persistent = -1

    ; Maximum number of links (persistent + non-persistent).  -1 means no limit.
    ; http://php.net/mysql.max-links
    mysql.max_links = -1

    ; Default port number for mysql_connect().  If unset, mysql_connect() will use
    ; the $MYSQL_TCP_PORT or the mysql-tcp entry in /etc/services or the
    ; compile-time value defined MYSQL_PORT (in that order).  Win32 will only look
    ; at MYSQL_PORT.
    ; http://php.net/mysql.default-port
    mysql.default_port =3306

    ; Default socket name for local MySQL connects.  If empty, uses the built-in
    ; MySQL defaults.
    ; http://php.net/mysql.default-socket
    mysql.default_socket=/var/run/mysqld/mysqld.sock

    ; Default host for mysql_connect() (doesn't apply in safe mode).
    ; http://php.net/mysql.default-host
    mysql.default_host =

    ; Default user for mysql_connect() (doesn't apply in safe mode).
    ; http://php.net/mysql.default-user
    mysql.default_user =

    ; Default password for mysql_connect() (doesn't apply in safe mode).
    ; Note that this is generally a *bad* idea to store passwords in this file.
    ; *Any* user with PHP access can run 'echo get_cfg_var("mysql.default_password")
    ; and reveal this password!  And of course, any users with read access to this
    ; file will be able to reveal the password as well.
    ; http://php.net/mysql.default-password
    mysql.default_password =

    ; Maximum time (in seconds) for connect timeout. -1 means no limit
    ; http://php.net/mysql.connect-timeout
    mysql.connect_timeout = 60

    ; Trace mode. When trace_mode is active (=On), warnings for table/index scans and
    ; SQL-Errors will be displayed.
    ; http://php.net/mysql.trace-mode
    mysql.trace_mode = Off

    [MySQLi]

    ; Maximum number of persistent links.  -1 means no limit.
    ; http://php.net/mysqli.max-persistent
    mysqli.max_persistent = -1

    ; Allow accessing, from PHP's perspective, local files with LOAD DATA statements
    ; http://php.net/mysqli.allow_local_infile
    ;mysqli.allow_local_infile = On

    ; Allow or prevent persistent links.
    ; http://php.net/mysqli.allow-persistent
    mysqli.allow_persistent = On

    ; Maximum number of links.  -1 means no limit.
    ; http://php.net/mysqli.max-links
    mysqli.max_links = -1

    ; If mysqlnd is used: Number of cache slots for the internal result set cache
    ; http://php.net/mysqli.cache_size
    mysqli.cache_size = 2000

    ; Default port number for mysqli_connect().  If unset, mysqli_connect() will use
    ; the $MYSQL_TCP_PORT or the mysql-tcp entry in /etc/services or the
    ; compile-time value defined MYSQL_PORT (in that order).  Win32 will only look
    ; at MYSQL_PORT.
    ; http://php.net/mysqli.default-port
    mysqli.default_port = 3306

    ; Default socket name for local MySQL connects.  If empty, uses the built-in
    ; MySQL defaults.
    ; http://php.net/mysqli.default-socket
    mysqli.default_socket=/var/run/mysqld/mysqld.sock

    ; Default host for mysql_connect() (doesn't apply in safe mode).
    ; http://php.net/mysqli.default-host
    mysqli.default_host =

    ; Default user for mysql_connect() (doesn't apply in safe mode).
    ; http://php.net/mysqli.default-user
    mysqli.default_user =

    ; Default password for mysqli_connect() (doesn't apply in safe mode).
    ; Note that this is generally a *bad* idea to store passwords in this file.
    ; *Any* user with PHP access can run 'echo get_cfg_var("mysqli.default_pw")
    ; file will be able to reveal the password as well.
    ; http://php.net/mysqli.default-pw
    mysqli.default_pw =

    ; Allow or prevent reconnect
    mysqli.reconnect = Off

    [mysqlnd]
    ; Enable / Disable collection of general statistics by mysqlnd which can be
    ; used to tune and monitor MySQL operations.
    ; http://php.net/mysqlnd.collect_statistics
    mysqlnd.collect_statistics = On

    ; Enable / Disable collection of memory usage statistics by mysqlnd which can be
    ; used to tune and monitor MySQL operations.
    ; http://php.net/mysqlnd.collect_memory_statistics
    mysqlnd.collect_memory_statistics = Off

    ; Size of a pre-allocated buffer used when sending commands to MySQL in bytes.
    ; http://php.net/mysqlnd.net_cmd_buffer_size
    ;mysqlnd.net_cmd_buffer_size = 2048

    ; Size of a pre-allocated buffer used for reading data sent by the server in
    ; bytes.
    ; http://php.net/mysqlnd.net_read_buffer_size
    ;mysqlnd.net_read_buffer_size = 32768

    [bcmath]
    ; Number of decimal digits for all bcmath functions.
    ; http://php.net/bcmath.scale
    bcmath.scale = 0

    [mcrypt]
    ; not needed since PHP has mcrypt compiled into it
    ; extension=mcrypt.so

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
    #       # NOTE: You should have "cgi.fix_pathinfo = 0;" in php.ini
    #
    #       # With php5-cgi alone:
    #       fastcgi_pass 127.0.0.1:9000;
    #       # With php5-fpm:
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
