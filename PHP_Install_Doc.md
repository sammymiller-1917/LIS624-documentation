# PHP Installation Documentation

## Overview

This document records the process I took in installing PHP on my Ubuntu VM hosted on Google Cloud.
The goal was to verify that PHP was correctly installed and configured using a browser/OS detection script.

## Environment:

- VM Provider: Google Cloud
- OS: Ubuntu 22.04 LTS
- Web Server: Apache 2.4.52
- PHP Version: 8.1
- Browser Used for testing: Safari (Mac)

## Step 1: Update Package Lists

First, I updated my package list to make sure that I was up-to-date and installing the latest available version of PHP.

I ran `sudo apt update` and `sudo apt upgrade`.

This was completed without errors.

## Step 2: Installing PHP

Since I already had Apache installed from a previous instance of work in the VM (see `03-05_Documentation_update.md` for documentation of that process). As a result, I only needed to install PHP and make sure it was interfacing with Apache correctly.

I ran the following commands:
```
sudo apt install php libapache2-mod-php
sudo systemctl restart apache2
```

The first command installed php, the latter created a connection between PHP and my existing Apache web server.

I initially misspelled the `sudo apt install php libapache2-mod-php` command, and the command line interface returned an error. This was resolved by double checking the spelling (making sure to input `libapache2-mod-php` rather than `libapache-mod-php`.)

Then, I ran the command `php -v` to confirm the installed version of PHP.

Then, I ran the command `systemctl status apache2` to check the status of Apache. Apache was enabled properly. Initially, I did not know how to exit out of the output the CLI produced for the `systemctl status apache2` command. I had to repeatedly restart my VM instance to exit out of the command, before realizing that pressing the q key exits out of the output.

## Step 3: Checking PHP Installation

I first navigated to the Apache root directory and created a file called `info.php` using the following commands:
```
cd /var/www/html/
sudo nano info.php
```

I then wrote the following code into the `info.php` file:
```
<?php
phpinfo();
?>
```

Then, to confirm that the `info.php` file was created successfully, I navigated to http://34.170.249.97/info.php in my browser (currently Safari, because at home I am using a hand-me-down MacBook Air).

There were no errors in the creation of the `info.php` file nor in navigating to the page on my Apache-hosted website.

After confirming the file was created and I was able to navigate to it, and that the page displayed correct information, I deleted `info.php`.

## Step 4: Configuring Apache and PHP

I used the following commands to navigate to the proper directory and edit Apache's `dir.conf` file, after making a backup just in case:
```
cd /etc/apache2/mods-available/
sudo cp dir.conf dir.conf.bak
sudo nano dir.conf
```

Then, I inserted `index.php` into the `DirectoryIndex` in the `dir.conf` file, so that `index.php` was first in the line of index files.

I then ran `apachectl configtest`, which returned `Syntax OK`. After this, I reloaded apache using `sudo systemctl reload apache2` and checked the status wit h`systemctl status apache2`. It was at this point that I actually learned how to exit the output of `systemctl` using the q key.

## Step 5: Creating an index.php File

I navigated back to the Apache root directory with `cd /var/www/html` and used `sudo nano index.php` to create an editable `index.php` file.

Then I copy-pasted the code specified in the lecture notes (normally I make a point of typing code out by hand. This was the only instance of me copy/pasting anything during this installation process.

The code in question:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Browser Detector</title>
</head>
<body>
    <h1>Browser & OS Detection</h1>
    <p>You are using the following browser to view this site:</p>

    <?php
    $user_agent = $_SERVER['HTTP_USER_AGENT'];

    // Browser Detection
    if (stripos($user_agent, 'Edg') !== false || stripos($user_agent, 'Edge') !== false) {
        $browser = 'Microsoft Edge';
    } elseif (stripos($user_agent, 'Firefox') !== false) {
        $browser = 'Mozilla Firefox';
    } elseif (stripos($user_agent, 'Chrome') !== false && stripos($user_agent, 'Chromium') === false) {
        $browser = 'Google Chrome';
    } elseif (stripos($user_agent, 'Chromium') !== false) {
        $browser = 'Chromium';
    } elseif (stripos($user_agent, 'Opera Mini') !== false) {
        $browser = 'Opera Mini';
    } elseif (stripos($user_agent, 'Opera') !== false || stripos($user_agent, 'OPR') !== false) {
        $browser = 'Opera';
    } elseif (stripos($user_agent, 'Safari') !== false && stripos($user_agent, 'Chrome') === false) {
        $browser = 'Safari';
    } else {
        $browser = 'Unknown Browser';
    }

    // OS Detection
    if (stripos($user_agent, 'Windows') !== false) {
        $os = 'Windows';
    } elseif (stripos($user_agent, 'iOS') !== false || stripos($user_agent, 'iPhone') !== false || stripos($user_agent, 'iPad') !== false) {
        $os = 'iOS';
    } elseif (stripos($user_agent, 'Android') !== false) {
        $os = 'Android';
    } elseif (stripos($user_agent, 'Mac') !== false || stripos($user_agent, 'Macintosh') !== false) {
        $os = 'Mac';
    } elseif (stripos($user_agent, 'Linux') !== false) {
        $os = 'Linux';
    } else {
        $os = 'Unknown OS';
    }

    // Output Result
    echo "<p>Your browser is <strong>$browser</strong> and your operating system is <strong>$os</strong>.</p>";
    ?>

</body>
</html>
```

This code is the browser detection script mentioned above. It uses PHP to, when the webpage is visited, output a result telling the user/visitor their OS and web browser, rather than the webpage content in the `index.html` file.

After saving and exiting `nano`, I visited my website homepage, and confirmed that the browser/OS detection script was working as intended.

## Issues Encountered

The only major issues were initially misspelling the command to install PHP (`sudo apt install php libapache2-mod-php`), and initially not knowing how to exit the `systemctl` command output. Otherwise there were no errors or issues.

## Reflection

I learned that installing programs is straightforward when you actually spell them correctly, and that verifying the status of programs is important but has its own distinct methods of navigation from the normal CLI. I also reiterated my existing awareness of the elevated permissions for modifying the Apache root directory and any of the directories involved in configuring Apache. I learned to restart services after installing them, although I did not attempt to see what would happen if I did not restart Apache, etc.

I did this documentation in one go immediately after going through the installation and configuration of PHP, rather than writing it as I went along. This process was relatively easy, but I would like to perform documentation as I go along the next time I work on something. This will require having multiple terminal windows open, which I was not aware was an option prior to now.

## Results

PHP is installed and configured to interface with my preexisting Apache install. The browser detection script confirmed proper server-side execution.
