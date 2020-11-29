# PHP7-from-scratch

## Instructions on how to compile PHP 7.4.6 from source on Ubuntu 20.04 machines

This will also include instructions on how to configure the machine for Nginx.

These instructions are ideal for those who want to run PHP in production.
This version of PHP lacks intrusive packages like Suhosin.

And added benefit, is you don't have install any packages like
`php-json`, `php-mysql`, `php-common`, nor `php-cli`

Instead, you have 1 PHP binary, and a PHP FPM binary.

## Step 1: Create directory for the PHP Binaries

The PHP binaries will be installed in a `bin` directory off of the user's
home directory.

    mkdir -p ~/bin/

## Step 2: Add the bin directory to your Path

This will allow you to use type `php` anywhere and the version of PHP we're
compiling will be first on the list - and the first one to be run

Open the file `~/.bashrc` and add the following to the end of it

    cat >> ~/.bashrc

Now, paste the following into your terminal

    # add local bin directory
    if [ -d "$HOME/bin/php7/bin" ] ; then
      PATH="$HOME/bin/php7/bin:$PATH"
    fi

    # press ctrl-c to exit cat

## Step 3: Install packages to prepare our system

Install the following packages. These are needed for PHP to communicate with
MySQL, Postgres, composer, and Nginx.

    sudo apt-get install autoconf build-essential \
      libssl-dev pkg-config zlib1g-dev libargon2-dev \
      libsodium-dev libcurl4-openssl-dev sqlite3 libsqlite3-dev \
      libonig-dev

## Install nGinx
If you want to use nginx instead of laravel's `artisan serve` or PHP's local web
server

    sudo apt-get install nginx

### Install MySQL

If you do not plan on installing MySQL, then _skip this step_

    sudo apt-get install mariadb-server

### install Postgres

If you don't plan on installing PostgreSQL then _skip this step_

    # I like to add a postgres user (but, this is optional)
    sudo adduser postgres

    sudo apt-get install postgresql-9.6 postgresql-client-9.6 \
    postgresql-contrib-9.6 libpq-dev

### Fix easy.h error __IF__ you do not install `pkg-config`

__IF__ you installed the package `pkg-config` _skip this step_

Debian based distros don't report the correct location of
openSSL headers. Without the `pkg-config` package PHP looks in the wrong
location for the headers. This symlink fixes that.

    sudo ln -s /usr/include/x86_64-linux-gnu/curl /usr/include/curl

## Step 4: Download Latest PHP 7.4.x

As of the time of this, PHP 7.4.6 was the latest PHP available.

__NOTE: this installs PHP in the user's local bin directory, not
globally!__

    # create the Download directory
    mkdir -p ~/Downloads/

    # change to the Downloads directory and fetch the PHP 7.3 tarball
    cd ~/Downloads;
    wget https://www.php.net/distributions/php-7.4.6.tar.bz2 -O php-7.4.6.tar.bz2
    tar -xf php-7.4.6.tar.bz2
    cd php-7.4.6


## Step 5: Build a build script

This is our base build script. Create a file called `build_php.sh` and paste
this snippet into it


    #!/bin/sh

    ######################
    # To build PHP
    # STEP 1: sh build_php.sh
    # STEP 2: make -j number_of_cores_or_processors_CPU_has
    # STEP 3: make install
    ######################

    INSTALL_DIR=$HOME/bin/php7

    mkdir -p $INSTALL_DIR

    ./configure --prefix=$INSTALL_DIR \
        --enable-bcmath \
        --enable-fpm \
        --with-fpm-user=www-data \
        --with-fpm-group=www-data \
        --disable-cgi \
        --enable-mbstring \
        --enable-shmop \
        --enable-sockets \
        --enable-sysvmsg \
        --enable-sysvsem \
        --enable-sysvshm \
        --with-zlib \
        --with-curl \
        --without-pear \
        --with-openssl \
        --enable-pcntl \
        --with-password-argon2 \
        --with-sodium \
        --disable-libxml \
        --disable-simplexml \
        --disable-xml \
        --disable-xmlreader \
        --disable-xmlwriter \
        --disable-dom \
        --without-libxml 

    # to add additional compile options to PHP, add " \" after the last
    # element. You *must* put a space and then a backslash so the configure
    # script knows that the there is more to the build script



### IF YOU INSTALL MARIADB, ADD THIS TO THE FOLLOWING LINES FROM THIS SCRIPT

    --enable-mysqlnd \
    --with-pdo-mysql=mysqlnd

### IF YOU INSTALL POSTGRES, ADD THIS THE FOLLOWING LINES FROM THIS SCRIPT

    --with-pdo-pgsql=/usr/bin/pg_config


### IF YOU NEED TO WORK WITH ZIP FILES

Install the ZIP libraries if you need to work with .zip file

Earlier we installed the package `zlib1g-dev`, this allows us to work with
compressed files, like tar.gz and .tgz, but not .zip files.

    sudo apt-get install libzip-dev libzip5

And, add this to the build PHP build script below

    --with-zip \


### TO USE PsySH - A PHP repl useful for debugging Laravel apps

    sudo apt-get install libreadline-dev

And, add the following to the PHP build script below

    --with-readline \

### IF YOU NEED TO USE XML FILES

    sudo apt-get install libxml2-dev

*remove* the following lines from the PHP build script below

    --disable-libxml \
    --disable-simplexml \
    --disable-xml \
    --disable-xmlreader \
    --disable-xmlwriter \
    --disable-dom

## Step 6: Compile PHP

The next three commands create a Makefile so the `make` command knows how to
build PHP, where to install the binaries at, what options PHP should be built
with

The variable `number_of_cores_or_processors_CPU_has` should be at least 3 unless
you are building on a small box, or a VM, however, this increases the time
greatly

    sh build_php.sh
    make -j number_of_cores_or_processors_CPU_has
    make install

## Step 7: Edit the PHP.ini file

Copy the *php-7.3.4/php.ini-development* file from the source directory to
the *~/bin/php7/lib/* directory, and rename the file to *php.ini*

    cp php.ini-development ~/bin/php7/lib/php.ini
    cd ~/bin/php7/lib;

## Step 8: Edit the php.ini file to change your sessings

### Change your timezone

You'll need to set your timezone to one of the following values from here:
[http://php.net/manual/en/timezones.php](http://php.net/manual/en/timezones.php)

For example, if you are in the US Central timezone, your timezone will be
 `America/Chicago`

If you are in the US Pacific Timezone, your timezone will be `America/Los_Angeles`

If you are in Zurich, Switzerland, your timezone will be `Europe/Zurich`

    [Date]
    ; Defines the default timezone used by the date functions
    ; http://php.net/date.timezone
    date.timezone=America/Chicago

The MySQL socket PDO connects to _(if MySQL is running locally)_

    ; Default socket name for local MySQL connects.  If empty, uses the built-in
    ; MySQL defaults.
    ; http://php.net/pdo_mysql.default-socket
    pdo_mysql.default_socket=/var/run/mysqld/mysqld.sock

The PHP-FPM fix

    cgi.fix_pathinfo=0:

## Step 8: PHP-FPM Config Files

You need to create a *php-fpm.conf* file to use Nginx.

    cd ~/bin/php7/etc/; mv php-fpm.conf.default php-fpm.conf
    cd ~/bin/php7/etc/php-fpm.d/; mv www.conf.default www.conf

In your favorite editor, open the www.conf file, and change the user and group
from nobody to www-data (the user for nginx). **THIS SHOULD BE DONE FOR YOU
JUST DOUBLE CHECK TO MAKE SURE IT'S SET**

    ...
    user = www-data
    group = www-data
    ...

### IF YOU WANT TO USE SOCKETS INSTEAD OF TCP PORT 9000

Sockets are faster then using TCP ports, but are limited in the number of
connections they support.

If you want to use sockets, though, you'll need to make the following changes
to the `www.conf` file

change `listen` from

    listen = 127.0.0.1:9000
to

    listen = /var/run/php-fpm.sock

also, uncomment out the `listen.ower`, `listen.group`, & `listen.mode` variables

    listen.owner = www-data
    listen.group = www-data
    listen.mode = 0660


## Step 9: ADD PHP TO YOUR PATH

Edit your _.bashrc_ file and add the PHP 7 bin directories to your path

    if [ -d "$HOME/bin/php7" ] ; then
      PATH="$HOME/bin/php7/bin:$HOME/bin/php7/sbin:$PATH"
    fi

## Step 10: Download Composer

    cd ~/bin
    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php composer-setup.php
    ln -s composer.phar composer

## Step 11: Installing & Configuring XDebug

Do this for local development, and for testing servers

### Run PHP-FPM at Boot Time
To run the PHP-FPM process whenever the server reboots run the following command

    sudo crontab -eu root

Then add the following line to the crontab

    @reboot /home/me/bin/php-fpm

# XDEBUG

### Download and un-tar the XDebug Module
_DO *NOT* install XDebug on a production server!_


    wget https://xdebug.org/files/xdebug-2.9.6.tgz -O xdebug-2.9.6.tgz
    tar -xzf xdebug-2.9.6.tgz

### Compile the XDebug Module

The variable number_of_cores_or_processors_CPU_has should be at least 3. Most
modern computers have quad core processors. This significantly speeds up the
time taken to compile the PHP binary and its shared libraries

    cd xdebug-2.9.6/
    phpize
    ./configure --enable-xdebug
    make -j number_of_cores_or_processors_CPU_has
    make install


The _XDebug_ installation script **should** install _xdebug.so_ in the correct location which is
_$HOME/bin/php7/lib/php/extensions/no-debug-non-zts-20190902/_

Test if the file exists

    ls -l $HOME/bin/php7/lib/php/extensions/no-debug-non-zts-20190902/xdebug.so

If you get an error stating **no such file or directory**, then, you need to
move the shared object to the PHP extension directory manually

    cp modules/xdebug.so $HOME/bin/php7/lib/php/extensions/no-debug-non-zts-20190902/xdebug.so


## Step 12: Add xdebug setting to the PHP.ini file

You need to tell PHP where to find the `xdebug` library. Append this snippet
to the end of the php.ini file located at __$HOME/bin/php7/lib/php.ini__

    [xdebug]
    zend_extension=/home/{USER NAME}/bin/php7/lib/php/extensions/no-debug-non-zts-20190902/xdebug.so
    xdebug.remote_enable=1
    xdebug.remote_autostart=1
    xdebug.remote_connect_back=1
    xdebug.remote_host=127.0.0.1
    xdebug.remote_port=9000



## Step 13: Using PHP with Nginx (production mode)

You'll have to start *php-fpm* manually to get it working with Nginx

    sudo $HOME/bin/php7/sbin/php-fpm

### Running PHP-FPM at Boot Time

To run the PHP-FPM process whenever the server reboots run the following command

    sudo crontab -eu root

Then add the following line to the crontab

    @reboot /home/me/bin/php-fpm

### Configure Nginx to Use PHP-FPM

You need to modify Nginx's configuration to start using PHP-FPM

    cd /etc/nginx/sites-available

Open *default* and change the root directory to where your code is

    root /path/to/your_PHP_project;

Add index.php to the index list

    index index.php;

Remove the *location* section and make the following changes in the
FastCGI section

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        # If you want to use a TCP port, un-comment this
        # fastcgi_pass 127.0.0.1:9000;

        # ...OR
        # If you want to use Unix sockets, un-comment this
        # fastcgi_pass unix:/var/run/php-fpm.sock;
    }

### Example Nginx PHP-FPM configuration File

So, your conf file should look like this:

    server {
        listen 80 default_server;
        listen [::]:80 default_server;

        # path to code (index.php should be in this directory)
        root /path/to/PHPCODE;

        index index.php;

        server_name phpdevbox;

        location / {
          try_files $uri $uri/ /index.php?query_string;
        }

        location ~ \.php$ {
            include snippets/fastcgi-php.conf;

            # use this if you want to use TCP:
            # fastcgi_pass 127.0.0.1:9000;

            # use this if you want to use sockets
            # fastcgi_pass unix:/var/run/php-fpm.sock;
        }

        location ~ /\.ht {
          # deny access to .htaccess files
          deny all;
        }
    }

Nginx should now be serving your files

## STEP 13: Running PHP's Built in Server (development mode)

There are multiple ways to use PHP for development.

### Laravel 5.5

If you're using Laravel 5.5 this is how you start the built in PHP server

    php artisan serve --host=your.ip.of.vm --port=8000

### PHP CLI SERVER

If you want to use the built in PHP web-server

    php -S ip.of.your.machine:port_number

### Nginx

While, you can use Nginx for testing, it's not the best alternative. One of the
issue you will find is that Nginx times out.

Follow the instructions found in the section
__Using PHP with Nginx (production mode)__ on how to setup Nginx to serve PHP
files from your VM

#### Nginx Virtualbox Bug

You'll need to edit `/etc/nginx/nginx.conf` and change the line
`sendfile on;` to `sendfile off;`

What will happen is changes to static files (CSS and Javascript) files won't
be updated. Turning off _sendfile_ will cause Nginx to serve the file via a
different method and the new file's changes will be displayed immediately.

# Development On Remote Server

These step are if you want to develop [write code] on Windows or Mac,
but run the code on a VM.

I prefer Ubuntu Server because it lacks a window manager and only needs about 256 MB RAM

Every step in the below instructions has been tailored specifically for
Ubuntu Server on Virtualbox.

## VIRTUALBOX
After installing Ubuntu Server (20.04) in a virtual machine, install the following packages

    nfs-kernel-server

NFS (Network File System) is targeted at *NIX based systems. Originally developed by Sun
MicroSystems in the 1980s, NFS allows the file system of one *NIX computer to be
accessed by another *NIX system.

## Create an NFS Mount
First create a directory to be shared. It's best to use a directory with no assigned
user by default

   mkdir -p /mnt/shared_dir

Each folder to be shared via NFS must have an entry in the file _/etc/exports_

    # NOTE: NO SPACE AFTER THE IP ADDRESS!
    /mtn/shared_dir client_ip(permissions)

 * The first arguement is pretty simple: the directory to share.
 * The second argument is the client's IP or IP range: e.g. _192.168.1.1/24_. 
   That IP address means anyone from 192.168.1.1 - 192.168.1.255. If you want 
   anyone to be able to connect, change the IP Address to *
 * The third arguement is the permissions

## NFS Permissions
Here's the list of permissions that work on Mac/Ubuntu to an Ubuntu server

 * rw
 * sync
 * no_subtree_check
 * insecure
 * all_squash - _tells NFS that for any user connecting,ignore their actual 
   user & group ID and treat them as if user ID = anonuid and group ID = anonguid_
 * anonuid=1000
 * anonguid=1000

**IF** _anonuid_ and _anonguid_ are 0,then that user **IS GIVEN ROOT PERMISSION**

The _anonuid_ and _anonuid_ of 1000 is the user and group of the user on my
Ubuntu Server

## Restart the NFS Deamon
Now that we have a shared dir, allow others to connect to it

    sudo exportfs -a
    sudo systemctl restart nfs-kernel-server

## Connect to the NFS share
Using MacOS Catalina (10.15) this is one way to connect

    cd # change to the home directory
    mkdir -p nfs_mount # create a new directory called "nfs_mount"
    sudo mount 192.168.1.184:/mnt/shared_dir /Users/{my mac user}/nfs_mount

## Connect IDE to Remote Server
If you are using VSCode this _launch.json_ will allow you to write code locally
and debug remotely

    {
      "version": "0.2.0",
      "configurations": [
        {
            "name": "Listen for XDebug",
            "type": "php",
            "request": "launch",
            "port": 9000,
            "pathMappings": {
                "/home/phpuser/cifs_share": "${workspaceRoot}"
            }
        }
      ]
    }