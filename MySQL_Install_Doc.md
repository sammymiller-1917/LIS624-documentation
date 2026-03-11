# MySQL Installation Documentation

## Overview

This document records the process I took in installing and configuring MySQL on my Ubuntu VM hosted on Google Cloud. The goal is to configure MySQL, PHP, and Apache such that renders `opacdb` and `books` data on the Apache-hosted webpage.

## Environment

- VM Provider: Google Cloud
- OS: Ubuntu 22.04 LTS
- Web Server: Apache 2.4.52
- PHP Version: 8.1
- Browsers Used for Testing: Safari (Mac), Firefox (Windows)

## Step 1: Update Package Lists

First, I updated my package list to make sure that I was up-to-date and installing the latest available version of MySQL.

I ran `sudo apt update`, `sudo apt upgrade`, `sudo apt autoremove`, and `sudo apt clean`.

This was completed without errors.

## Step 2: Installing MySQL

To install MySQL, I first ran the command `sudo apt install mysql-server`. When prompted to select Y or n, I pressed Y.

## Step 3: Configuring MySQL

After letting the installation process run, I began to configure MySQL.

First, I confirmed the version number with `mysql --version`. This confirmed I have mysql  Ver 8.0.45-0ubuntu0.24.04.1 for Linux on x86_64 ((Ubuntu)) installed.

Then, I ran the command `systemctl status mysql` to confirm that MySQL was running and enabled. The output confirmed this.

Following this, I ran the command `sudo mysql_secure_installation`, which sets up a secure configuration for MySQL. Following the prompts provided by the command output, I selected `Y` to validate passwords, remove anonymous users, disallow root login remotely, remove test database and access to it, and reload privilege tables. I selected 0 for LOW on the settings for the password validation policy. In a normal configuration, it would be important to select a higher level of policy security, but for our purposes here LOW is acceptable.

Then I ran the command `sudo mysql -u root`, which logs into the MySQL database. By using the command `show databases;` while logged into MySQL, I was able to see the existing databases in the install of MySQL, displayed below:

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.03 sec)
```

While logged into the root user in MySQL, I set up the `opacuser` user with the following command: `create user 'opacuser'@'localhost' identified by 'XXXXXXXXXXX';`, with the Xs standing in for the original password. This returned `Query OK`, thus indicating the user was set up successfully.

## Step 3: Creating a Practice Database

While still logged into the root user in MySQL, I used the following commands to create a new database set up so `opacuser` has total access to it.

```
create database opacdb default character set utf8mb4 collate utf8mb4_0900_ai_ci;
show databases;
grant all privileges on opacdb.* to 'opacuser'@'localhost';
```

The first command creates the database and sets the character encoding to UTF-8 in order to support international characters. `show databases` shows all of the extant databases, confirming the presence of `opacdb` among them. `grant all privileges on opacdb.* to 'opacuser'@'localhost';` does as stated, giving full privileges to the `opacuser` to create, delete, edit, insert, etc. files within the database `opacdb`.

With this practice database created, I logged out of the root user.

## Step 4: Logging in as opacuser and Creating Tables

First, before logging back into MySQL as `opacuser`, I opened `~/.bashrc` in nano to eit the MySQL command prompt and make it more informative. To do this, I inserted the line `export MYSQL_PS1="[\d]>"` into the file at the bottom. After this, I logged back into MySQL with `mysql -u opacuser -p`, which prompted me to input the password.

Once logged in, I used the following commands:

```
show databases;
use opacdb;
```

The former command lists the available databases, which only displays the following:

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| opacdb             |
| performance_schema |
+--------------------+
3 rows in set (0.00 sec)
```

The latter command output `Database changed`. 

I then ran the following command to create a table called `books` with four fields for information on a book (`id`, `author`, `title`, and `copyright`). `id` functions as the primary key in the table and set to be a whole number, while `author` and `title` are set to have a maximum length of 150 characters and that they should not be empty. The line `copyright` is set to the `year` data type, which stores four-digit years and also should not be empty.

Running the command `show tables;` outputs the following:

```
+------------------+
| Tables_in_opacdb |
+------------------+
| books            |
+------------------+
1 row in set (0.09 sec)
```

Running the command `describe books;` outputs the following:

```
+-----------+--------------+------+-----+---------+----------------+
| Field     | Type         | Null | Key | Default | Extra          |
+-----------+--------------+------+-----+---------+----------------+
| id        | int unsigned | NO   | PRI | NULL    | auto_increment |
| author    | varchar(150) | NO   |     | NULL    |                |
| title     | varchar(150) | NO   |     | NULL    |                |
| copyright | year         | NO   |     | NULL    |                |
+-----------+--------------+------+-----+---------+----------------+
4 rows in set (0.03 sec)
```

With the table configuration confirmed, I used the following command to insert data on four books into the `books` table:

```
insert into books (author, title, copyright) values
('Jennifer Egan', 'The Candy House', '2022'),
('Imbolo Mbue', 'How Beautiful We Were', '2021'),
('Lydia Millet', 'A Children\'s Bible', '2020'),
('Julia Phillips', 'Disappearing Earth', '2019');
```

Then, with the `select * from books;` command, I was able to see the `books` table with the data properly displayed:

```
+----+----------------+-----------------------+-----------+
| id | author         | title                 | copyright |
+----+----------------+-----------------------+-----------+
|  1 | Jennifer Egan  | The Candy House       |      2022 |
|  2 | Imbolo Mbue    | How Beautiful We Were |      2021 |
|  3 | Lydia Millet   | A Children's Bible    |      2020 |
|  4 | Julia Phillips | Disappearing Earth    |      2019 |
+----+----------------+-----------------------+-----------+
4 rows in set (0.01 sec)
```

## Step 5: Testing MySQL Commands

I used the following commands with the following outputs:

Command: `select author from books;`

Output:
```
+----------------+
| author         |
+----------------+
| Jennifer Egan  |
| Imbolo Mbue    |
| Lydia Millet   |
| Julia Phillips |
+----------------+
4 rows in set (0.00 sec)
```

Command: `select copyright from books;`

Output:
```
+-----------+
| copyright |
+-----------+
|      2022 |
|      2021 |
|      2020 |
|      2019 |
+-----------+
4 rows in set (0.00 sec)
```

Command: `select author, title from books;`

Output:
```
+----------------+-----------------------+
| author         | title                 |
+----------------+-----------------------+
| Jennifer Egan  | The Candy House       |
| Imbolo Mbue    | How Beautiful We Were |
| Lydia Millet   | A Children's Bible    |
| Julia Phillips | Disappearing Earth    |
+----------------+-----------------------+
4 rows in set (0.00 sec)
```

Command: `select author from books where author like '%millet%';`

Output:
```
+--------------+
| author       |
+--------------+
| Lydia Millet |
+--------------+
1 row in set (0.01 sec)
```

Command: `select title from books where author like '%mbue%';`

Output:
```
+-----------------------+
| title                 |
+-----------------------+
| How Beautiful We Were |
+-----------------------+
1 row in set (0.11 sec)
```

Command: `select author, title from books where title not like '%e';`

Output:
```
+----------------+--------------------+
| author         | title              |
+----------------+--------------------+
| Julia Phillips | Disappearing Earth |
+----------------+--------------------+
1 row in set (0.01 sec)
```

Command: `select * from books;`

Output:
```
+----+----------------+-----------------------+-----------+
| id | author         | title                 | copyright |
+----+----------------+-----------------------+-----------+
|  1 | Jennifer Egan  | The Candy House       |      2022 |
|  2 | Imbolo Mbue    | How Beautiful We Were |      2021 |
|  3 | Lydia Millet   | A Children's Bible    |      2020 |
|  4 | Julia Phillips | Disappearing Earth    |      2019 |
+----+----------------+-----------------------+-----------+
4 rows in set (0.01 sec)
```

Commands:
```
alter table books add publisher varchar(75) after title;
describe books;
update books set publisher='Simon & Schuster' where id='1';
update books set publisher='Penguin Random House' where id='2';
update books set publisher='W. W. Norton & Company' where id='3';
update books set publisher='Knopf' where id='4';
select * from books;
```

Output:
```
+-----------+--------------+------+-----+---------+----------------+
| Field     | Type         | Null | Key | Default | Extra          |
+-----------+--------------+------+-----+---------+----------------+
| id        | int unsigned | NO   | PRI | NULL    | auto_increment |
| author    | varchar(150) | NO   |     | NULL    |                |
| title     | varchar(150) | NO   |     | NULL    |                |
| publisher | varchar(75)  | YES  |     | NULL    |                |
| copyright | year         | NO   |     | NULL    |                |
+-----------+--------------+------+-----+---------+----------------+
5 rows in set (0.03 sec)

+----+----------------+-----------------------+------------------------+-----------+
| id | author         | title                 | publisher              | copyright |
+----+----------------+-----------------------+------------------------+-----------+
|  1 | Jennifer Egan  | The Candy House       | Simon & Schuster       |      2022 |
|  2 | Imbolo Mbue    | How Beautiful We Were | Penguin Random House   |      2021 |
|  3 | Lydia Millet   | A Children's Bible    | W. W. Norton & Company |      2020 |
|  4 | Julia Phillips | Disappearing Earth    | Knopf                  |      2019 |
+----+----------------+-----------------------+------------------------+-----------+
4 rows in set (0.00 sec)
```

Command: `delete from books where author='Julia Phillips';`

Output: `Query OK, 1 row affected (0.01 sec)`

Confirmation: ran `select * from books;`
```
+----+---------------+-----------------------+------------------------+-----------+
| id | author        | title                 | publisher              | copyright |
+----+---------------+-----------------------+------------------------+-----------+
|  1 | Jennifer Egan | The Candy House       | Simon & Schuster       |      2022 |
|  2 | Imbolo Mbue   | How Beautiful We Were | Penguin Random House   |      2021 |
|  3 | Lydia Millet  | A Children's Bible    | W. W. Norton & Company |      2020 |
+----+---------------+-----------------------+------------------------+-----------+
3 rows in set (0.00 sec)
```

Command:
```
insert into books
(author, title, publisher, copyright) values
('Emma Donoghue', 'Room', 'Little, Brown & Company', '2010'),
('Zadie Smith', 'White Teeth', 'Hamish Hamilton', '2000');
select * from books;
```

Output:
```
+----+---------------+-----------------------+-------------------------+-----------+
| id | author        | title                 | publisher               | copyright |
+----+---------------+-----------------------+-------------------------+-----------+
|  1 | Jennifer Egan | The Candy House       | Simon & Schuster        |      2022 |
|  2 | Imbolo Mbue   | How Beautiful We Were | Penguin Random House    |      2021 |
|  3 | Lydia Millet  | A Children's Bible    | W. W. Norton & Company  |      2020 |
|  5 | Emma Donoghue | Room                  | Little, Brown & Company |      2010 |
|  6 | Zadie Smith   | White Teeth           | Hamish Hamilton         |      2000 |
+----+---------------+-----------------------+-------------------------+-----------+
5 rows in set (0.12 sec)
```

Command: `select author, publisher from books where copyright < '2011';`

Output:
```
+---------------+-------------------------+
| author        | publisher               |
+---------------+-------------------------+
| Emma Donoghue | Little, Brown & Company |
| Zadie Smith   | Hamish Hamilton         |
+---------------+-------------------------+
2 rows in set (0.01 sec)
```

Command: `select author from books order by copyright;`

Output:
```
+---------------+
| author        |
+---------------+
| Zadie Smith   |
| Emma Donoghue |
| Lydia Millet  |
| Imbolo Mbue   |
| Jennifer Egan |
+---------------+
5 rows in set (0.00 sec)
```

Following the above, I quit MySQL with the `\q` command.

## Step 6: Installing PHP/MySQL Support

In order to complete the connection between PHP and MySQL for the purpose of using both on the Apache-hosted website, I first ran the command `sudo apt install php-mysql` to install some modules to support the interaction between the two programs.

Following this, I ran `sudo systemctl restart apache2` and `sudo systemctl restart mysql` to restart both Apache and MySQL.

Then, I ran the following commands to create and edit a `login.php` file in the `/var/www` directory in order so that MySQL can authenticate itself with PHP.

```
cd /var/www
sudo touch login.php
sudo chmod 640 login.php
sudo chown :www-data login.php
ls -l login.php
sudo nano login.php
```

With the `login.php` file open in nano, I inserted the following code into the file, with the Xs once again standing in for the password I set earlier:

```
<?php // login.php
$db_hostname = "localhost";
$db_database = "opacdb";
$db_username = "opacuser";
$db_password = "XXXXXXXXX";
?>
```

Then, navigating over to `/var/www/html` with the command `cd /var/www/html`, I used `sudo nano opac.php` to create a file thusly named and input the following code into it:

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MySQL Server Example</title>
</head>
<body>

    <h1>A Basic OPAC</h1>
    <p>We can retrieve all the data from our database and book table using a couple of different queries.</p>

    <?php
    // Load MySQL credentials securely
    require_once '/var/www/login.php';

    // Enable detailed MySQL error reporting
    mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);

    // Establish database connection
    $conn = new mysqli($db_hostname, $db_username, $db_password, $db_database);

    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }

    echo "<h2>Query 1: Retrieving Publisher and Author Data</h2>";

    // Query using prepared statement
    $stmt = $conn->prepare("SELECT publisher, author FROM books");
    $stmt->execute();
    $result = $stmt->get_result();

    while ($row = $result->fetch_assoc()) {
        echo "<p>Publisher " . htmlspecialchars($row["publisher"]) .
             " published a book by " . htmlspecialchars($row["author"]) . ".</p>";
    }

    $stmt->close();

    echo "<h2>Query 2: Retrieving Author, Title, and Date Published Data</h2>";

    $stmt2 = $conn->prepare("SELECT author, title, copyright FROM books");
    $stmt2->execute();
    $result2 = $stmt2->get_result();

    while ($row = $result2->fetch_assoc()) {
        echo "<p>A book by " . htmlspecialchars($row["author"]) .
             " titled <em>" . htmlspecialchars($row["title"]) .
             "</em> was released in " . htmlspecialchars($row["copyright"]) . ".</p>";
    }

    $stmt2->close();
    $conn->close();
    ?>

</body>
</html>
```

In order to test that both of these .php files are functioning without errors, I ran the command `sudo php -f` on both filepaths (i.e. `sudo php-f /var/www/login.php` and `sudo php -f /var/www/html/opac.php`) This indicated that both files had no errors.

At this point, I attempted to visit my webpage as indicated in the lecture notes, but the simple OPAC did not display on the homepage (instead it still displayed the OS/browser detection script from the previous week). As it turned out, however, I misread the lecture notes and the simple OPAC displaying at opac.php rather than on the homepage was correct and functioning as intended.

## Issues Encountered

The only issues encountered were due to user error; accidental mistypes, misreads of the lecture notes leading to panic over nothing, etc. Ultimately, all of the above commands were executed successfully without needing much modification or correction from simple mistakes on my part (I only mention this owing to one misspelling of a record in `books`, which was swiftly corrected).

## Reflection

I learned what MySQL is (of the three components of our LAMP stack (Apache, PHP, and MySQL), I was the least familiar with MySQL), and how to use and navigate MySQL, namely both accessing the MySQL root and setting up a regular user account. I also learned how to create a relational database in MySQL and populate it with data. I am beginning to see where this will go as we continue to work toward the final project of creating a library website! Working with the `books` data in MySQL felt the closest to home of any of our work yet, very similar to populating an existing database with metadata from item records. I also learned how to connect this data with PHP in order to display it on a webpage.

## Conclusion

MySQL is installed, enabled, functions properly, and is set up to interface with PHP in order to display content on the Apache-hosted website. The regular user `opacuser` is set up with MySQL, and a basic library OPAC has been set up to display on the Apache-hosted website.
