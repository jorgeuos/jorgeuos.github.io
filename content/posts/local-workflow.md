---
title: "Get up and running on MacBook localhost quickly ⚡️"
date: 2023-01-12T14:24:11+01:00
draft: false
tags: ["apache","localhost","php","wordpress","wp","brew"]
categories: ["Development"]
---

# Easy way to get up and running on MacBook localhost

Sometimes I just want to get started real quickly with a new project. So instead spending hours or even days configuring docker images and so on and instead of repeating everything step-by-step I try to automate and document everything in case I forget what I did, which I do all the time.

Sometimes I need to be able to work without any internet connection e.g. so localhost suits the best for that, so I don't get stalled by not being able to download new images.


## Install Apache with brew

If you haven't installed Apache with brew, perhaps you want to. I don't really remember why I do it, but it's been like that for ages.

```sh
sudo apachectl stop
sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
brew install httpd
# Double check you don't have anything running on port 80.
sudo lsof -i :80
brew services start httpd
brew services list
```

## Prepare your local dev env

```sh
mkdir -p /usr/local/var/www/mylocal.dev
echo "My mylocal.dev site works!" > /usr/local/var/www/mylocal.dev/index.html
```


## Add a Vhost file to your local setup

```sh
cd /usr/local/etc/httpd/extra
touch mylocal.dev.conf
```

For a WP site e.g. or any other php application, copy paste the following into `mylocal.dev.conf`:

```sh
# You might want to Listen to any other port
# Make sure that you Listen to it too.
# Listen 80
<VirtualHost *:80>
    ServerAdmin you@example.com
    DocumentRoot "/usr/local/var/www/mylocal.dev"

    ServerName mylocal.dev

    <IfModule dir_module>
      DirectoryIndex index.html index.php
    </IfModule>
    <FilesMatch \.php$>
        SetHandler application/x-httpd-php
    </FilesMatch>

    <Files ".ht*">
        # Require all denied
        Require all granted
    </Files>

    <Directory "/usr/local/var/www/mylocal.dev">
      # If you want experiment with some different headers
      # Header set Access-Control-Allow-Origin "mylocal.dev"
      # Header add Access-Control-Allow-Origin "127.0.0.1"
      # Header set Access-Control-Allow-Origin *
      Header set Access-Control-Allow-Origin "*"
      Header set Access-Control-Allow-Headers "Content-type"
      Header set Content-Security-Policy "default-src 'self'; style-src 'self' 'unsafe-inline' data:; font-src 'self' 'unsafe-inline' 'unsafe-eval' data:; img-src 'self' data:; script-src 'self' 'unsafe-inline' 'unsafe-eval'; connect-src 'self';"

      Options +Indexes +MultiViews +FollowSymLinks
      AllowOverride All
      Require all granted
    </Directory>

    ErrorLog "/usr/local/var/log/httpd/mylocal.dev-error.log"
    CustomLog "/usr/local/var/log/httpd/mylocal.dev-access.log" common
</VirtualHost>
```

At some point, I started to import my vhosts 1-by-1 for whatever reason.
Just make sure that you're including the conf in your `/usr/local/etc/httpd/httpd.conf` file.
```
echo "Include /usr/local/etc/httpd/extra/mylocal.dev.conf" >> /usr/local/etc/httpd/httpd.conf
```

## Great!! Now we can take it for a spin and try it out.

Check syntax and activate your local site:
```sh
sudo httpd -t
```

If you get any errors, a good command to use is:
```sh
httpd -e error
```

Remember to add your local domain to our `/etc/hosts`
```sh
sudo bash -c 'echo "127.0.0.1 mylocal.dev" >> /etc/hosts'
```

### Everything is fine and dandy?!

```
brew services restart httpd
```

Test if it works:
```sh
curl http://mylocal.dev
My mylocal.dev site works!

# Checkout our headers and follow redirects if any:
curl -IL http://mylocal.dev
My mylocal.dev site works!
```

### Success!

## Perhaps you want to set up a WP site

Create new DB in your local MySQL. If you don't have any, install it by running `brew install mysql`.
If you bump into any errors, make sure you don't have any old versions of mysql running:
```sh
brew remove mysql
brew cleanup
launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
rm ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
sudo rm -rf /usr/local/var/mysql
brew install mysql
```

Alternative way of starting it:
```sh
unset TMPDIR
mysql_install_db --verbose --user=`whoami` --basedir="$(brew --prefix mysql)" --datadir=/usr/local/var/mysql --tmpdir=/tmp
/usr/local/Cellar/mysql/5.5.10/bin/mysql_secure_installation
#start
launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
#stop
launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
```

Log in to mysql:
```sh
mysql -uroot -p
Enter password:
# Enter your password
```

Create DB and user and grant permissions:
```sql
CREATE DATABASE `local` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci */ /*!80016 DEFAULT ENCRYPTION='N' */
CREATE USER 'local'@'localhost' IDENTIFIED WITH authentication_plugin BY 'password';
GRANT ALL PRIVILEGES ON `*`.`local` TO 'local'@'localhost' WITH GRANT OPTION;
```

Done!


## Download and install WP

Make sure you have [wp-cli](https://wp-cli.org/) installed, it will make your life much easier.
```
cd /usr/local/var/www/mylocal.dev/
wp core download
wp config create --dbname=local --dbuser=local --dbpass=password
wp core install --url="mylocal.dev" --title="My local site" --admin_user="admin" --admin_email="you@example.com"
# Admin password: #1337113711371337
# Success: WordPress installed successfully.
```

