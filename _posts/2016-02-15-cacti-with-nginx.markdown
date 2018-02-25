---
layout: post
title:  "Setting up Cacti 0.8.8 on Ubuntu 14.04 with nginx"
date:   2016-02-15 00:00:00
keywords:
  - cacti
  - nginx
  - ubuntu
  - setup
  - guide
  - php-fpm
  - fastcgi
---

I recently installed and set up a [Cacti][cacti] graphing server on my Ubuntu 14.04 LTS headless server with [nginx][nginx] as my chosen web server.
I've decided to share the basic configuration steps that I took.

## Cacti with nginx

1. Install all pre-requisite packages for Cacti

   * `apt-get install nginx snmpd php5-fpm`
   * Note that if you do not plan on using Apache as your web server (as in my case), you must install the [php5-fpm][pkg-php5-fpm]
     package *before* installing packages [php5][pkg-php5] and [cacti][pkg-cacti], otherwise apt will choose to install the
     [libapache2-mod-php5][pkg-libapache2-mod-php5] Apache 2 module as a dependency for them.
     This package is unnecessary if you don't have Apache installed.

1. Edit your nginx config file (usually found under `/etc/nginx/` directory) and
   [enable handling of php scripts via nginx's fastcgi directives][nginx-fastcgi-php].
   Don't forget to add "index.php" to the index directive within the php location block context.

1. Edit php-fpm's [www.conf][php-fpm-www.conf] file

   * Make php use a unix socket instead of a tcp socket by setting the `listen` directive to `/var/run/php5-fpm.sock`
   * Set `listen.owner` and `listen.group` directives to the same user and group that your web server is running as
   * Set `listen.mode` directive to `0660`

1. Install php5, cacti and the spine poller

   * `apt-get install php5 php5-cgi cacti cacti-spine`

1. Edit your nginx config file once again to add in a cacti location block

   ```nginx
   location /cacti {
       alias /usr/share/cacti/site/;
       index index.php;
       gzip off;
   
       location ~ \.php$ {
           fastcgi_split_path_info ^(.+\.php)(/.+)$;
           include fastcgi_params;
           fastcgi_pass unix:/var/run/php5-fpm.sock;
       }
   }
   # Disallow access to the cacti scripts directory
   # to prevent requests to download script files
   location /cacti/scripts {
       deny all;
       return 404;
   }
   ```

1. Alternatively, instead of using the nginx alias directive you can make a symlink to the cacti static files
   (though I prefer using the alias directive described above)

   * Change directory to your nginx server root directory (`/usr/share/nginx/www` by default)
   * `ln -s /usr/share/cacti/site cacti`

1. Restart nginx and php5-fpm services to allow their configuration changes to take affect

1. Load up the cacti web page which should now be running on your local web server at `http://localhost/cacti`
   and complete the one-time setup. Default cacti login is admin/admin.

   * Change the poller type under the settings page to "spine"
   * Change the poller interval to every minute
   * Under "System Utilities", click on "Rebuilt Poller Cache"

## References

[Installing the Cacti Server Monitor on Ubuntu 12.04 Cloud Server](https://www.digitalocean.com/community/tutorials/installing-the-cacti-server-monitor-on-ubuntu-12-04-cloud-server)

[How to install Cacti with Nginx](http://notblog.org/cacti-with-nginx/)

## Resetting Cacti entirely

1. Stop the poller (Cacti settings > Poller tab)

1. Delete all rrd files

   `rm -rf /var/lib/cacti/rra/*.rrd`

1. Drop/rename cacti database from MySQL

   `mysql -u root -pPASSWORD -S /run/mysqld/mysqld.sock -h localhost -N -e "DROP DATABASE cacti;"`

1. Reconfigure cacti by following the prompts

   `dpkg-reconfigure cacti`

[cacti]: http://www.cacti.net/
[nginx]: https://www.nginx.com/resources/wiki/
[pkg-php5-fpm]: http://packages.ubuntu.com/trusty/php5-fpm
[pkg-php5]: http://packages.ubuntu.com/trusty/php5
[pkg-cacti]: http://packages.ubuntu.com/trusty/cacti
[pkg-libapache2-mod-php5]: http://packages.ubuntu.com/trusty/libapache2-mod-php5
[nginx-fastcgi-php]: http://askubuntu.com/a/134676
[php-fpm-www.conf]: http://php.net/manual/en/install.fpm.configuration.php
