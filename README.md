Apache+PHP with Phing Build Pack
================================

This is a build pack bundling PHP and Apache for Heroku apps built with Phing. The standard [PHP build pack](https://github.com/heroku/heroku-buildpack-php) takes your PHP source as is and deploys it directly to Apache, but this build pack runs a customized [Phing](http://www.phing.info/) build during the Git push before deploying to Apache. This is helpful for more complex PHP apps that require code generation or other preprocessing to prepare source code for deployment. 


Usage
-----

Repositories should have a Phing build file called `build.xml` in their root directory with a `stage` target that deposits web-accessible files in the directory designated by the `PUBLIC_HTML_DIR` environment variable. Projects may also deposit files that should not be accessed by users in the directory designated by the `PRIVATE_DIR` environment variable. For example, here is a minimal `build.xml` that simply copies the source from the `src/main/php` directory for deployment:

    <project name="Sample Phing Build" default="www">
        <target name="stage">
            <fail unless="env.PUBLIC_HTML_DIR" message="Enivronment variable PUBLIC_HTML_DIR must be set"/>
            <copy todir="${env.PUBLIC_HTML_DIR}">
                <fileset dir="src/main/php"/>
            </copy>
        </target>
    </project>

Of course, most builds would be more complex with file manipulations, mappings, or filtering. The `PUBLIC_HTML_DIR` and `PRIVATE_DIR` environment variables are also available at runtime for access by PHP scripts.


Configuration
-------------

The config files are bundled with the build pack itself:

* conf/httpd.conf
* conf/php.ini


Pre-compiling binaries
----------------------

    # apache
    mkdir /app
    curl --location -O http://www.gtlib.gatech.edu/pub/apache//httpd/httpd-2.2.22.tar.gz
    tar xvzf httpd-2.2.22.tar.gz 
    cd httpd-2.2.22
    ./configure --prefix=/app/apache --enable-rewrite --enable-expires --enable-headers --disable-setenvif     
    make
    make install
    cd ..
    
    # php
    wget http://us2.php.net/get/php-5.3.15.tar.gz/from/us.php.net/mirror 
    mv mirror php.tar.gz
    tar xzvf php.tar.gz
    cd php-5.3.15/
    ./configure --prefix=/app/php --with-apxs2=/app/apache/bin/apxs --with-mysql --with-pdo-mysql --with-pgsql --with-pdo-pgsql --with-iconv --with-gd --with-curl=/usr/lib --with-config-file-path=/app/php --enable-soap=shared --with-openssl --enable-cli --with-readline --enable-pcntl --with-mcrypt  --enable-shared=no --enable-static=yes
    make
    make install
    cd ..
    
    # php extensions
    mkdir /app/php/ext
    cp /usr/lib/libmysqlclient.so.16 /app/php/ext/
    cp /usr/lib/libmcrypt.so.4 /app/php/ext/
    
    # package
    cd /app
    echo '2.2.22' > apache/VERSION
    tar -zcvf apache-2.2.22.tar.gz apache
    echo '5.3.15' > php/VERSION
    tar -zcvf php-5.3.15.tar.gz php

Meta
----

Created by Pedro Belo and extended for Phing by Ryan Brainard.
Many thanks to Keith Rarick for the help with assorted Unix topics :)
