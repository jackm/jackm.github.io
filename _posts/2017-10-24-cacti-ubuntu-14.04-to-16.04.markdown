---
layout: post
title:  "Upgrading Cacti with nginx after upgrading Ubuntu 14.04 to 16.04"
date:   2017-10-24 20:57:00
keywords:
  - cacti
  - nginx
  - ubuntu
  - setup
  - guide
  - upgrade
  - Ubuntu 16.04
  - PHP 7.0
  - fastcgi
---

I recently upgraded headless Ubuntu server from Trusty 14.04 LTS to Xenial 16.04 LTS and also had to deal with getting Cacti working again after the change. The package repo for Xenial 16.04 comes with Cacti 0.8.8f where as the repo for Trusty 14.04 came with Cacti 0.8.8b.

Here are the additional steps that I had to take to get Cacti working again with nginx after upgrading Ubuntu versions.

1. You should be safe to let apt update the `cacti` and `cacti-spine` packages without fear of losing and cacti user data; but it is always a good idea to [backup the cacti MySQL database and backup the main cacti directory][cacti-backup] at `/usr/share/cacti/`.

1. Right after the upgrade to 16.04 was complete, I noticed that when I tried to load my cacti http page I got an HTTP 500 server error!
   I knew this must have something to do with how nginx was passing php script execution off to the backend php-fpm server since this error only showed up when I tried to load the cacti http page.
   Even other php pages were working just fine.

   Turns out this was due to the alias directive in conjuntion with trying to pass php execution off to a backend.
   Because my cacti site files are located in a different directory than my web server root directory, I need to modify the `SCRIPT_FILENAME` fastcgi variable so that php-fpm knows the correct filepath it is supposed to execute.
   This is explained by the creator of nginx in [this email thread][nginx-fastcgi-alias].

   Also note that nginx actually has bundled with it [two fastcgi config files][nginx-fastcgi-history], with only one slight difference between the two!

   In short, I had to add `fastcgi_param SCRIPT_FILENAME $request_filename;` in my cacti php location block.

1. Xenial 16.04 has moved to PHP 7.0 instead of PHP 5.
   The `php5-*` and `php5-fpm` packages are gone and replaced with their `php-7.0-*` counterparts. The new `php7.0-fpm` package changes the filename for the php-fpm socket.
   This means that you'll have to edit your nginx configs and change the `fastcgi_pass` directive to point to the new socket filename. By default this is `/var/run/php/php7.0-fpm.sock`.

   Change all nginx config lines from `fastcgi_pass unix:/var/run/php5-fpm.sock;` to `fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;`.

   Many thanks to [Chris' blog post][nginx-php] for these instructions.

1. Another consequence of moving to PHP 7.0 is that the `poller.php` script at `/usr/share/cacti/site/poller.php` will break.
   This is noticable since all of the graphs will show NaN values and there will be errors shown in `/var/log/cacti/poller-error.log`

   This is because in upgrading from PHP 5 to PHP 7.0, the [split()][php-split] function was removed, and there are two palces in poller.php where split() is used.

   A [fix has been implemented for this][cacti-bugfix], but as of right now it has not yet been added to the main xenial archive so apt will not see the new package hotfix.
   The new cacti package version is `0.8.8f+ds1-4ubuntu4.16.04.3` where as the version that comes in the standard xenial archives is `0.8.8f+ds1-4ubuntu4.16.04.2` (.2 instead of .3).
   Right now the updated package only exists in the `xenial-proposed` archive.
   This -proposed archive can be added by following the [instructions on the Ubuntu wiki][ubuntu-proposed].
   Once this is done, and the apt cache is updated via `apt-get update`, you can force the install of the newer cacti package by running `sudo apt-get install cacti=0.8.8f+ds1-4ubuntu4.16.04.3`.
   This version of the package updates poller.php (and probably other things too) so that it will run with PHP 7.0.

   `cacti` package version table before adding -proposed archive:

   ```
   user@server:~$ apt-cache policy cacti
   cacti:
     Installed: 0.8.8f+ds1-4ubuntu4.16.04.2
     Candidate: 0.8.8f+ds1-4ubuntu4.16.04.2
     Version table:
    *** 0.8.8f+ds1-4ubuntu4.16.04.2 500
           500 http://archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages
           500 http://archive.ubuntu.com/ubuntu xenial-updates/universe i386 Packages
           500 http://archive.ubuntu.com/ubuntu xenial-security/universe amd64 Packages
           500 http://archive.ubuntu.com/ubuntu xenial-security/universe i386 Packages
        0.8.8f+ds1-4ubuntu4 500
           500 http://archive.ubuntu.com/ubuntu xenial/universe amd64 Packages
           500 http://archive.ubuntu.com/ubuntu xenial/universe i386 Packages
   ```

   And after adding the -proposed archive and installing the newer package version:

   ```
   user@server:~$ apt-cache policy cacti
   cacti:
     Installed: 0.8.8f+ds1-4ubuntu4.16.04.3
     Candidate: 0.8.8f+ds1-4ubuntu4.16.04.3
     Version table:
    *** 0.8.8f+ds1-4ubuntu4.16.04.3 500
           500 http://archive.ubuntu.com/ubuntu xenial-proposed/universe amd64 Packages
           500 http://archive.ubuntu.com/ubuntu xenial-proposed/universe i386 Packages
           100 /var/lib/dpkg/status
        0.8.8f+ds1-4ubuntu4.16.04.2 500
           500 http://archive.ubuntu.com/ubuntu xenial-updates/universe amd64 Packages
           500 http://archive.ubuntu.com/ubuntu xenial-updates/universe i386 Packages
           500 http://archive.ubuntu.com/ubuntu xenial-security/universe amd64 Packages
           500 http://archive.ubuntu.com/ubuntu xenial-security/universe i386 Packages
        0.8.8f+ds1-4ubuntu4 500
           500 http://archive.ubuntu.com/ubuntu xenial/universe amd64 Packages
           500 http://archive.ubuntu.com/ubuntu xenial/universe i386 Packages
   ```

1. You also probably want to make sure that cacti is the only package from the xenial-proposed archive that apt will try to install; ignoring all other packages from -proposed in favour of the ones from the 'standard' archives.
   To do this, you can use [package pinning][package-pinning].

   Create a file in the `/etc/apt/preferences.d/` directory (doesn't really matter what it's called) and set the pin priority of all packages within the xenial-proposed archive to something lower (eg. 400 instead of the default 500), and then pin the `cacti` package back to priority 500. Now, cacti will be the only package from xenial-proposed that apt will try to install/update.

   ```
   Package: *
   Pin: release a=xenial-proposed
   Pin-Priority: 400
   
   Package: cacti
   Pin: release a=xenial-proposed
   Pin-Priority: 500
   ```

1. Cacti 0.8.8f now depends on jquery to show the graph tree and display graphs (This appears to have been added since [version 0.8.8c][cacti-changelog]).
   The `javascript-common` package provides jquery files under `/usr/share/javascript/`, however if your nginx root directory is not `/usr/share/` then nginx won't be able to find these files.
   Cacti will try to request jquery files at `http(s)://example.com/javascript/...`.
   When using nginx we need to make some additional config settings so that nginx can find and serve these jquery files.

   You now must add the following location block so that nginx can serve jquery; otherwise you won't be able to see the graph tree or any graphs while in "Tree view".

   ```nginx
   location /javascript {
           alias /usr/share/javascript/;
   }
   ```

   The whole cacti specific config will now look like the following:

   ```nginx
   # Include this file on your nginx.conf to support Cacti graphing server
   location /cacti {
           alias /usr/share/cacti/site/;
           index index.php;
           gzip off;
   
           location ~ \.php$ {
                   # NOTE: You should have "cgi.fix_pathinfo=0" in php.ini
   
                   include fastcgi_params;
                   fastcgi_param SCRIPT_FILENAME $request_filename;
                   fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
           }
   }
   
   # Cacti uses jquery from the javascript directory provided by javascript-common package
   location /javascript {
           alias /usr/share/javascript/;
   }
   
   # Disallow access to the cacti scripts directory
   # to prevent requests to download script files
   location /cacti/scripts {
           deny all;
           return 404;
   }
   ```

1. Don't forget to reload nginx so that all of the config changes take effect!

   ```
   sudo nginx -t
   sudo nginx -s reload
   ```

As a final thought, I am also considering upgrading to Cacti version 1.x, however since the Xenial repos probably won't ever get a version higher than 0.8.8 it means that I'll likely have to install Cacti manually and not have the luxurity of having `apt` manage the package for me.
This is something I'll tackle on a future date. :grimacing:

[cacti-backup]: https://www.cacti.net/downloads/docs/html/upgrade.html
[nginx-fastcgi-history]: https://blog.martinfjordvald.com/2013/04/nginx-config-history-fastcgi_params-versus-fastcgi-conf/
[nginx-fastcgi-alias]: https://forum.nginx.org/read.php?2,3059,3060#msg-3060
[nginx-php]: https://heald.ca/upgrade-ubuntu-nginx-php5-14-04-trusty-16-04-xenial/
[php-split]: http://php.net/manual/en/function.split.php
[cacti-bugfix]: https://bugs.launchpad.net/ubuntu/+source/cacti/+bug/1662027
[ubuntu-proposed]: https://wiki.ubuntu.com/Testing/EnableProposed
[package-pinning]: https://help.ubuntu.com/community/PinningHowto
[cacti-changelog]: https://www.cacti.net/changelog.php
