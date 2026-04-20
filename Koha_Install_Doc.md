# Koha ILS/OPAC Installation Documentation

## Overview
This document records the process I took in installing Koha, an open source ILS (Integrated Library System), for use with the example library website we are building as the final project in the LIS 624 course.

We will primarily be using Koha for a public-facing OPAC, which provides access to the library catalog, namely via a search function.

## Environment

- VM Provider: Google Cloud
- OS: Ubuntu 24.04 LTS
- Web Server: Apache 2.4.52
- PHP Version: 8.3.6
- MySQL Version: Ver 8.0.45
- WordPress Version: 6.9.4
- Omeka Version: 3.2

## Step 1: Setting up a New Virtual Machine Instance

Our Koha install has to be on a new VM because there is simply not enough RAM on the original/primary VM for it to properly install and function.

I navigated to the "Create instance" page on Google Cloud, and set it to have the following settings and tags:

- Name: spring-2026-koha
- Region: us-central1 (Iowa)
- Machine type: 32 medium (2 vCPU, 1 core, 4 GB memory)
- OS and storage: Ubuntu 24.04 LTS, 20 GB Hard Disk
- Networking: check off "Allow HTTP traffic"
- Network Tags: koha-staff-8080 and koha-opac-8081

Those latter configurations open up the VM's firewall to HTTP traffic via the ports 8080 (for a staff interface) and 8081 (for a public interface). This is important once we try to connect the Koha ILS installation on this VM to the other, already extant VM.

Once those configurations were set, I clicked "Create" at the bottom of the page.

## Step 2: Setting Firewall Rules

To continue opening up/controlling HTTP traffic in the specific ways we need, we need to set certain rules for the VM's firewall for those specified ports.

While in the Google Cloud Console, I clicked on the hamburger icon in the top left, which brings up navigation in Google Cloud. From there, I navigated down to the tab labeled "VPC Network," and clicked on "Firewall."

On the firewalls page, I clicked on "Create a firewall rule," and set the following configurations:

- Name: koha-staff-8080
- Description: Open port 8080 for the Koha staff interface
- Target: Specified target tags
- Target tags: koha-staff-8080
- Source IPv4 ranges: allow access from anywhere with 0.0.0.0/0
- Specified protocols and ports: TCP: Ports: 8080

After setting the above configurations, I clicked "Create."

I then clicked "Create a firewall rule" again and set the following configurations for the second firewall rule:

- Name: koha-opac-8081
- Description: Open port 8081 for the Koha opac interface
- Target: Specified target tags
- Target tags: koha-opac-8081
- Source IPv4 ranges: allow access from anywhere with 0.0.0.0/0
- Specified protocols and ports: TCP: Ports: 8081

After setting the above configurations, I clicked "Create."

## Step 3: Installing Koha Repo

After opening the new VM (spring-2026-koha), I ran the command `tmux`, a terminal multiplexer that will allow a re-establishment of connection to the VM if the session is disconnected.

While in `tmux`, I ran `sudo apt update` and `sudo apt upgrade` to update my package lists and make sure everything is upgraded to its current versions. After that, I ran the command `sudo apt autoremove -y && sudo apt clean`, to clean up the installs from the previous commands.

### Sub-step 1: Adding Koha Repository

To add a new remote repository for Ubuntu to sync updates with, I ran the following commands:

```
sudo apt install apt-transport-https ca-certificates curl
sudo mkdir -p --mode=0755 /etc/apt/keyrings
sudo curl -fsSL https://debian.koha-community.org/koha/gpg.asc -o /etc/apt/keyrings/koha.asc
```

This sets signing keys to ensure that the VM installs authentic Koha software.

Next, I became the `root` user with the command `sudo su`.

In the `root` user, I copy/pasted the following commands into the terminal from the lecture notes:
```
tee /etc/apt/sources.list.d/koha.sources <<EOF
Types: deb
URIs: https://debian.koha-community.org/koha/
Suites: 25.05
Components: main
Signed-By: /etc/apt/keyrings/koha.asc
EOF
```

I used the command `cat /etc/apt/sources.list.d/koha.sources` to confirm that the contents matched the above inputs. Thankfully they did.

With the `exit` command, I returned to the regular user account.

## Step 4: Installing MariaDB

MariaDB is a MySQL fork which is one of two relational database systems that Koha can use. We are installing MariaDB because it is what Koha defaults to.

With the following commands, I installed MariaDB:
```
sudo apt update
sudo apt install mariadb-server
```

## Step 5: Installing Koha

Next, I needed to actually install Koha!

I ran `sudo apt update` to update the package lists in light of the Koha repository installation and MariaDB install.

Then, to receive the package information for Koha, I ran `apt show koha-common`. With the command `sudo apt install koha-common`, I install the Koha ILS.

## Step 6: Opening Ports

I needed to set up different ports for the staff and public interfaces of Koha, which we've already set up for some above.

First, I made a copy of the `koha-sites.conf` file with the command `sudo cp /etc/koha/koha-sites.conf /etc/koha/koha-sites.conf.backup`. This ensures that if I mess anything up we still have the original file.

Then, I opened `/etc/koha/koha-sites.conf` in nano and added the following text:
```
INTRAPORT="8080"
OPACPORT="8081"
```

## Step 7: Apache2 Setup

Thankfully, the Apache2 web server was automatically installed when we installed Koha, but we need to enable certain Apache modules for Koha to function properly. I used the following commands to do so:
```
sudo a2enmod rewrite cgi headers proxy_http
sudo systemctl restart apache2
```

After restarting Apache, I need to modify the web server to also interface with the ports we've designated. First, I make a copy of the `/etc/apache2/ports.conf` file as a backup.

Then, I opened `ports.conf` in nano and added the following two lines under where it said `Listen 80`:
```
Listen 8080
Listen 8081
```

## Step 8: Creating the Koha Instance

I used the following commands to create and initiate the install of Koha:
```
sudo koha-create --create-db bibliolib
sudo systemctl restart apache2
```

This output the following, indicating the process was completed successfully:
```
Koha instance is empty, no staff user created.
 * Starting Koha worker daemon for bibliolib (default)                                             [ OK ] 
 * Starting Koha worker daemon for bibliolib (long_tasks)                                          [ OK ] 
 * Starting Koha indexing daemon for bibliolib                                                     [ OK ]
```

## Step 9: Setting Additional Apache2 Configurations

Certain additional configurations need to be set to redirect Apache2's attentions away from the `/var/www/html` directory and toward the directories associated with Koha, as well as enabling network compression and enabling the new `bibliolib` library for Koha:
```
sudo a2dissite 000-default
sudo a2enmod deflate
sudo a2ensite bibliolib
```

After running the above commands, I ran `sudo systemctl reload apache2` to reload the new configurations and `sudo systemctl restart apache2` to restart the web server.

## Step 10: Using the Koha Web Installer

After using the command `sudo koha-passwd bibliolib` to get the password and username for the bibliolib library, I navigated to the VM's public IP in browser, with the staff port appended  (`http://34.68.168.47:8080`). This brought me to the Koha 25.05 web installer.

After entering the username and password received from `sudo koha-passwd bibliolib`, I followed the steps on the web installer, with the default options. When prompted to create an Administrator Identity, I set the username to testlibrarian and set a secure password. 

Once this was set up and I was able to log into the Koha staff/admin side and went into settings to set the `OPACBaseURL` in `System Preferences` to `http://34.61.150.36`

## Issues Encountered

Most issues were actually out of the computer, with several real life/personal events getting in the way of my completing the installation process in a timely manner. There were very few errors during the Koha install process. Most errors were quickly identified and resolved, such as one where I accidentally input incorrect information while setting up the firewall rules before attempting the actual install proper. I went in and edited the firewall rules in question in the Google Cloud command interface.

One issue which did occur was Apache2 crashing in between uses of Koha. After restarting, it seems to be working just fine right now. If it recurs, I will have to investigate further.

## Reflection

This was definitely the most involved installation we have done in this course, given it included work on the very backend through the Google Cloud interface itself, rather than just in the CLI or in the browser installation setup for the program. It took some thinking to wrap my head around some of how firewalls worked, but by following the instructions in the lecture notes I was able to fulfill the work satisfactorily. I applied skills learned in previous installs, especially for WordPress and Omeka, to this installation as well. As mentioned above, there were no issues that couldn't be resolved quickly or by a simple use of `sudo systemctl restart apache2`.

## Conclusion

Koha ILS is installed and configured. New firewall rules have been made to allow Koha to interface through the designated `8080` and `8081` ports. In addition, once this was all set up, I went in and added a button on the test library's website linking to the Koha OPAC. 
