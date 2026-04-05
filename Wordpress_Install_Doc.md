# Installing Wordpress

## Overview

This document records the process I took in installing Wordpress and configuring it for use as the front-facing website for a fictional library, as part of my coursework in the LIS 624 Systems Librarianship course.

Wordpress is an open source content management system (CMS) that is very widely used for building websites. By installing WordPress onto my VM and setting it up with Apache etc., I will be able to build a public-facing website using a widely standard CMS.

## Environment

- VM Provider: Google Cloud
- OS: Ubuntu 22.04 LTS
- Web Server: Apache 2.4.52
- PHP Version: 8.3.6
- MySQL Version: Ver 8.0.45

## Step 1: Installing WordPress from WordPress.org

After familiarizing myself with the installation guidelines on WordPress.org, I followed the custom installation guidelines provided in the lecture notes.

### Sub-step 1: Making sure that the VM's software matches WordPress's requirements

Since WordPress has more dependencies than most of the other software we have been installing, I need to make sure that PHP and MySQL are up-to-date to the minimum requirements for WordPress, which is PHP version 8.3 or greater and MySQL version 8.0 or greater.

Running `php --version` returns PHP 8.3.6. Running `mysql --version` returns MySQL Ver 8.0.45.

After this, I ran `sudo apt install php-curl php-xml php-imagick php-mbstring php-zip php-intl` to install additional PHP modules that allow WordPress to operate at full functionality.

The above command didn't run successfully no matter how many times I ran it. This made me unable to continue on the WordPress installation process for a few days until I could resolve the issue. After rebooting the system, running `sudo apt update` (which revealed I had 50 packages that were out of date), running `sudo apt upgrade` (which took about 30 minutes to upgrade everything), and then rebooting again, I ran the above command and everything installed properly.

Then, I ran `sudo systemctl restart apache2` to restart Apache and `sudo systemctl restart mysql` to restart MySQL.

### Sub-step 2: Downloading and Extracting WordPress

I first used the command `cd /var/www/html` to change to the /var/www/html directory, which houses all files used in running the Apache-hosted website, including HTML and PHP files.

Then, I used the command `sudo wget https://wordpress.org/latest.zip` to download the latest version of WordPress from wordpress.org as a .zip file.

Then, with the `unzip` command, I unzipped `latest.zip`. I did not have the `unzip` command installed, so I used the command `sudo apt install unzip` to install it before running the command again.

### Sub-step 3: Creating the WordPress Database and User

To use MySQL on a WordPress website, we need to make a new database and user to give WordPress access to MySQL.

To do that, I first log in as the MySQL root user with the command `sudo mysql -u root`. Then, I used the command `create user 'wordpress'@'localhost' identified by 'XXXXXXXXXXX';` (with the Xs being a password for the WordPress user.

I then created the new database with the command `create database wordpress;` and granted all privileges to the new wordpress user with `grant all privileges on wordpress.* to 'wordpress'@'localhost';`.

With the command `show databases;`, I confirmed that the new wordpress database was created. I exited MySQL with the command `\q`.

### Sub-step 4: Setting up wp-config.php

To give WordPress password access to the database, we navigate to the WordPress directory (with `cd /var/www/html/wordpress`) and edit the `wp-config.php` file.

I make a copy of `wp-config-sample.php` with the command `sudo cp wp-config-sample.php wp-config.php`.

Then, I opened `wp-config.php` in nano and inserted the wordpress database name at `DB_NAME`, the wordpress user name in `DB_USER`, and the password in `DB_PASSWORD`.

### Sub-step 5: Running the Install Script

Visiting my site's wordpress page at `34.170.249.97/wordpress` brings up the Welcome screen where I enter the site's information. I set the name to "Testville Public Library," the username to "test librarian", and set a strong password. After setting my email and clicking Install, WordPress was fully installed on my server!

## Issues Encountered

As noted above, the largest issue encountered was the persistent issue with installing the additional PHP modules required for properly installing WordPress. This made me unable to progress with finishing the install and configuration until I rebooted the system twice and ran `sudo apt update` and `sudo apt upgrade`. I initially panicked, and had to ask for assistance in the "Help!" module on the class page. Professor Burns was very helpful in encouraging me on where to look for what the issue was and how to fix it. Thankfully the fix was relatively simple.

No other issues were encountered.

## Reflection

I learned how to use WordPress on the developer side, how to integrate WordPress with the three components of our LAMP stack (Apache, PHP, and MySQL), and how to connect WordPress to a MySQL database.

I was already familiar with WordPress from the user end, and with blogs hosted on `wordpress.com`, but was completely unfamiliar with using WordPress on the developer side (i.e. the installation from `wordpress.org`). 

I also learned how to troubleshoot issues that presented serious roadblocks to my further progression, by using outside resources and attempting several fixes during troubleshooting until I found one that worked.

## Conclusion

WordPress is installed, PHP is updated and has additional modules installed to interface properly with WordPress, and a MySQL user and database have been created to interface with WordPress.
