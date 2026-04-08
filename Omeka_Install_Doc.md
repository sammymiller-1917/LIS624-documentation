# Omeka Classic Installation Documentation

## Overview

This document records the process I took in installing Omeka Classic, an open source web publishing platform for digital collections.

I have used Omeka in a prior course in the MLIS program, namely the Knowledge Organization course, so I am familiar with at least one side of the process of using Omeka, but not on the backend/installation side of things.

## Environment

- VM Provider: Google Cloud
- OS: Ubuntu 22.04 LTS
- Web Server: Apache 2.4.52
- PHP Version: 8.3.6
- MySQL Version: Ver 8.0.45
- WordPress Version: 6.9.4

## Step 1: Updates and Prerequisites

I first ran `sudo apt update` and `sudo apt upgrade` to update all of my packages. I then ran `sudo apt autoremove` and `sudo apt clean` to clean up any installations.

I checked installed modules in PHP with the command `php -m`, confirming that I do have the requisite modules for PHP installed for an Omeka installation (namely `mysqli` and `exif`).

I then installed ImageMagick with the command `sudo apt install imagemagick`. ImageMagick is a suite of utilities that allows Omeka to create thumbnail images of photos uploaded to the digital library, and otherwise interface with image files.

I then used the command `sudo a2enmod rewrite` to enable the Apache module `mod_rewrite`, allows Apache (and Omeka by extension) to rewrite URLs to more user-friendly versions.

This prompted me to use the command `sudo systemctl restart apache2` to restart Apache.

## Step 2: Installing and Configuring Omeka Classic

### Sub-step 1: Creating a New User and Database in MySQL

Omeka Classic, like WordPress, needs to use MySQL to function on the web server.

I logged into MySQL as the root user with the command `sudo mysql -u root`. Once logged in, I used the following commands to create a new user and new database:
```
create user 'omeka'@'localhost' identified by 'XXXXXXXXXXXXX';
create database omeka;
grant all privileges on omeka.* to 'omeka'@'localhost';
show databases;
```

These commands created a new user with the name `omeka` and a password indicated by the X's above, then created a new database also called `omeka`, and granted all privileges on the `omeka` database to the `omeka` user. With the command `show databases;`, I was able to confirm that the `omeka` database was properly created.

I quit out of MySQL with the `\q` command.

### Sub-step 2: Downloading and Extracting Omeka from Omeka.org

Navigating to the website `omeka.org/classic/download`, I was able to acquire the installation URL, `https://github.com/omeka/Omeka/releases/download/v3.2/omeka-3.2.zip`. 

After this, I navigated to `/var/www/html` with the command `cd /var/www/html`.

I then downloaded the latest version of Omeka Classic with the command `sudo wget https://github.com/omeka/Omeka/releases/download/v3.2/omeka-3.2.zip`, and, after the file downloaded onto the VM, unzipped it with the command `sudo unzip omeka-3.2.zip`.

This created a new directory in the `/var/www/html` directory called `omeka-3.2`. I renamed this with the command `sudo mv omeka-3.2. omeka`.

### Sub-step 3: Establishing Credentials in db.ini

With the `omeka` directory renamed, I navigated into it with the command `cd omeka`. Using the command `ls`, I could see that there was a file called `db.ini` in the directory.

I edited the `db.ini` file with `sudo nano db.ini`, and set the credentials in the file in order to allow Omeka Classic to interface with MySQL.

I set `host` to `localhost`, `username` to `omeka`, `password` to the same password as above, and `dbname` to `omeka`.

### Sub-step 4: Setting Apache Write Access

While still in the `omeka` directory, I ran the command `sudo chmod -R g+w *`. This gives Apache group write access on all files in the `omeka` directory.

After setting that permission, I restarted Apache with the command `sudo systemctl restart apache2`. I also ran `sudo systemctl restart mysql` just to be safe.

At this point, I encountered an issue. Despite enabling `mod_rewrite` above, when navigating to the omeka page on my website, it says there was an install error due to `mod_rewrite` not being enabled.

After a quick Google search, I was able to find the fix for the error. I navigated to the Apache root directory at `etc/apache2` and used `ls` to identify the file `apache2.conf`. With `sudo nano apache2.conf`, I edited the config file to read `AllowOverride All` under `/var/www/html`, the root directory for the Omeka install. This allows Apache to use `.htaccess` to override configuration directives, actually enabling `mod_rewrite` for Omeka.

### Sub-step 5: Finishing the Install Process

I navigated to Omeka on my site at `http://34.123.34.138/omeka/` and, after restarting Apache, was able to fill in the installation form in a similar way to WordPress, with a new username and password.

## Issues Encountered

There were only two issues I encountered. The first was actually unrelated to the install. I accidentally closed out of this very documentation file and had to completely restart my documentation, which set me back somewhat. The other was the above described issue with Omeka not being able to recognize when `mod_rewrite` was enabled. I was able to bypass this with the above described process whereby I enabled Override for All in `/var/www/html`.

## Reflection

I applied the same skills that I first learned in the WordPress installation process in installing Omeka. This installation process, thankfully, went much smoother than the WordPress installation (which took three days just to be able to install additional PHP modules correctly and then broke in a new way that stopped me from progressing on the Omeka install for a few days after that). The issue with Omeka not being able to recognize when `mod_rewrite` was enabled was relatively quickly resolved. I am becoming relatively adept at searching for fixes to issues online with Google and Stack Overflow. The "Help!" module and assistance I received from professor Burns were also incredibly important in me even being able to do this at all!

## Conclusion

Omeka Classic is installed and configured. I have not yet set up any contents for Omeka. In addition, a new MySQL user and database have been created to interface with Omeka Classic.
