---
layout: post
title:  "Setting up MNMP on Mac OSX Mavericks"
date:   2013-12-26 18:00:23
categories: blog
---

Starting requirements:

- Xcode ([Download from the Mac App Store](https://itunes.apple.com/de/app/xcode/id497799835))
- Xquarz
- Homewbrew

Note: The leading `$` is a placeholder for your terminal's prompt. You don't type it.


### Homebrew


Download Homebrew:

    $ ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"

Run the doctor to check all in well:

    $ brew doctor

Update & Upgrade Homebrew Formulas

    $ brew update
    $ brew upgrade

### PHP-FPM

Install PHP with homebrew:

    $ brew tap homebrew/dupes
    $ brew tap josegonzalez/homebrew-php

Run install

    $ brew install --without-apache --with-fpm --with-mysql php54 php54-mcrypt

Apache is a dependancy of PHP-FPM, so we pass `--without apache` to stop it being included. `--with-fpm` obviously includes the FPM module and `--with-mysql` includes the MySQL module.


If you wish to use this version of PHP in your terminal theh add the path to your .bash_profile file like so:

    $ echo 'export PATH="$(brew --prefix josegonzalez/php/php54)/bin:$PATH" >> ~/.bash_profile

Launch PHP-Fpm on startup

    $ mkdir -p ~/Library/LaunchAgents
    $ ln -sfv /usr/local/opt/php54/*.plist ~/Library/LaunchAgents

Start it now

    $ launchctl load -w ~/Library/LaunchAgents/homebrew-php.josegonzalez.php54.plist

Check its listening on port 9000,

    $ lsof -Pni4 | grep LISTEN | grep php
    
You should see something like:

    php-fpm   59104 User    6u  IPv4 0xc9098fd6af2dc9e9      0t0  TCP 127.0.0.1:9000 (LISTEN)
    php-fpm   59105 User    0u  IPv4 0xc9098fd6af2dc9e9      0t0  TCP 127.0.0.1:9000 (LISTEN)
    php-fpm   59106 User    0u  IPv4 0xc9098fd6af2dc9e9      0t0  TCP 127.0.0.1:9000 (LISTEN)
    php-fpm   59107 User    0u  IPv4 0xc9098fd6af2dc9e9      0t0  TCP 127.0.0.1:9000 (LISTEN)

### MariaDB

Install MariaDB with homebrew:

    $ brew install mariadb
    
To start MariaDB at login:

    $ ln -sfv /usr/local/opt/mariadb/*.plist ~/Library/LaunchAgents
    
Load MariaDB now:

    $ launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mariadb.plist
    
Now we need to run the MySQL database installer:

We need to find the bin correct location of the `my_print_defaults`, `mysql_secure_installation` and `mysql_install_db` files.

    $ which mysql_install_db
    $ which my_print_defaults
    $ which mysql_secure_installation
    
Neglecting the previous step may result in an error like:

`FATAL ERROR: Could not find ./bin/my_print_defaults`
    
If your bin location differs from `/usr/local/bin/` then use your own location instead.
    
Then run the command again with the new locations you got. Mine are entered below:
 
    $ unset TMPDIR
    $ /usr/local/bin/mysql_install_db --basedir=/usr/local 
    
Now run the secure instalation to setup your root password.

    $ /usr/local/bin/mysql_secure_installation
    
If you get an error at the beginning like `line 379: find_mysql_client: command not found` you can safely ignore it.

Following the defaults in the when promted should be fine.

Test your mysql connection:

	$ mysql -uroot -p
	
enter your root password you setup in the previous step and you shoudl see the MySQL Console:

	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

	MariaDB [(none)]>
	
You can exit the console by typing `\q` or `exit` then hitting `enter`. Or press `Ctrl + C`.

### Nginx

Install Nginx with homebrew;

	$ brew install nginx
	
Optional modules can be passed to compile like so:

	$ brew install nginx --with-http_ssl_module

A list of optional modules for nginx can be found [here](http://wiki.nginx.org/Modules).

Now we setup Nginx to start on login, however as we want Nginx to run on port 80 we must use the 'sudo' command.

	$ ln -sfv /usr/local/opt/nginx/*.plist ~/Library/LaunchAgents/
	$ chown root:wheel /usr/local/opt/nginx/homebrew.mxcl.nginx.plist
	
Start nginx right now:

	$ sudo launchctl load ~/Library/LaunchAgents/homebrew.mxcl.nginx.plist
	
*If the Firewall thats built into Mac OS is enabled; you may be promted weather or not to allow Nginx to the outside world. I chose to Allow it.*

Now lets check if Nginx is running on its default port of 8080:
	
	$ curl -IL http://localhost:8080
	
You should get an output like:

	HTTP/1.1 200 OK
	Server: nginx/1.4.4
	Date: Thu, 26 Dec 2013 03:27:27 GMT
	Content-Type: text/html
	Content-Length: 612
	Last-Modified: Thu, 26 Dec 2013 02:29:38 GMT
	Connection: keep-alive
	
Service Control:

At this point we start configuring everything therefore we need a simple way to start, stop and restart all the services. 

	$ curl https://gist.github.com/nicekiwi/8129472/raw/bash_aliases -o /tmp/.bash_aliases
	$ cat /tmp/.bash_aliases >> ~/.bash_aliases
	$ echo "source ~/.bash_aliases" >> ~/.bash_profile
	
Now you can use short aliases instead of typing in launchctl arguments and plist paths.

###### Nginx

You can start, stop and restart Nginx with:

	$ nginx.start
	$ nginx.stop
	$ nginx.restart
	
Check the Nginx `.conf` file.
	
	$ sudo nginx -t
	
###### PHP-FPM

Start, stop and restart PHP-FPM:

	$ php-fpm.start
	$ php-fpm.stop
	$ php-fpm.restart
	
Check the PHP-FPM `.conf` file.
	
	$ sudo php-fpm -t
	
###### MariaDB

	$ mysql.start
	$ mysql.stop
	$ mysql.restart

Now with all that sortred out we can setup our local server.
