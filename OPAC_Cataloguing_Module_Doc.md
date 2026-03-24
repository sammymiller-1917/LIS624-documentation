# Relational Databases, Barebones OPAC, and a Barebones Cataloguing Module

## Overview

This document records the process I took in setting up a new database in MySQL, using that new database in a barebones OPAC system, and setting up a barebones cataloguing module to further develop the ILS that we will be using in the final project of this course.

## Environment

- VM Provider: Google Cloud
- OS: Ubuntu 22.04 LTS
- Web Server: Apache 2.4.52
- PHP Version: 8.1
- MySQL Version: Ver 8.0.45

## Step 1: Update Package Lists

First, I updated my package list to make sure that I was up-to-date.

I ran `sudo apt upgrade`, `sudo apt autoremove`, and `sudo apt clean`.

This was completed without errors.

## Step 2: Creating a New Database and Tables

I logged into MySQL as the root user with the command `sudo mysql -u root`.

I then created a new database with `opacuser` having privileges in the database with the following commands (executed in the MySQL interface):

```
create database DinnerDB;
grant all privileges on DinnerDB.* to 'opacuser'@'localhost';
```

I then exited the root MySQL account with the `\q` command.

After logging back into MySQL as `opacuser` with the command `mysql -u opacuser -p`, I accessed the new database with the following commands:

```
show databases;
use DinnerDB;
```

I created a new table in the `DinnerDB` database called `Meals`, set to have five values: `meal_id`, `meal_name`, `cuisine`, `cooking_time`, and `vegetarian`. The first value is an integer and serves as the primary key for the relational database, while `meal_name` and `cuisine` containe variable-length strings, `cooking_time` is an integer with a CHECK constraint that stops the user from inputting a 0 or negative value, and `vegetarian` is a BOOLEAN true or false variable.

I used the following commands to create the above described table:
```
create table Meals (
	meal_id int auto_increment primary key,
	meal_name varchar(100) not null,
	cuisine varchar(50),
	cooking_time int not null default 1 check (cooking_time > 0),
	vegetarian boolean
);
```

I also created a new table called `Ingredients`, set to have four values: `ingredient_id`, `meal_id`, `ingredient_name`, and `quantity`. The first is the primary key, while `meal_id` is the primary key of `Meals` and here serves as a foreign key tying the two databases together, and both `ingredient_name` and `quantity` contain variable length strings.

I used the following commands to create the above described table:
```
create table Ingredients (
	ingredient_id int auto_increment primary key,
	meal_id int not null,
	ingredient_name varchar(100) not null,
	quantity varchar(50,
	foreign key (meal_id) references Meals(meal_id) on delete cascade
);
```

Using the command `show tables` while in the `DinnerDB` database, I was able to confirm that both tables were created successfully.

## Step 3: Inserting Data

I ran the following commands to insert four records into the `Meals` table:
```
insert into Meals (meal_name, cuisine, cooking_time, vegetarian) values
	('Spaghetti Bolognese', 'Italian', 45, FALSE),
	('Vegetable Stir Fry', 'Chinese', 20, TRUE),
	('Chicken Curry', 'Indian', 50, FALSE),
	('Mushroom Risotto', 'Italian', 35, TRUE);
```

Then, after using `select * from Meals;` to double check the `meal_id` values for the four meals, I used the following commands to insert records into the `Ingredients` table:
```
insert into Ingredients (meal_id, ingredient_name, quantity) values
	(1, 'Spaghetti', '200g'),
	(1, 'Ground Beef', '250g'),
	(1, 'Tomato Sauce', '1 cup'),
	(2, 'Broccoli', '100g'),
	(2, 'Carrots', '50g'),
	(2, 'Soy Sauce', '2T'),
	(3, 'Chicken Breast', '300g'),
	(3, 'Curry Powder', '2T'),
	(3, 'Coconut Milk', '1 cup'),
	(4, 'Arborio Rice', '1 cup'),
	(4, 'Mushrooms', '1 cup'),
	(4, 'Parmesan Cheese', '1/2 cup');
```

This took me three attempts, because of a syntax error in my first attempt owing to a mistyping, and then I unintentionally repeated the same syntax error mistype in my second attempt. Thankfully it was caught each time and I was able to repeat until I successfully populated the table with data.

## Step 4: Querying Data

To get practice using commands to query data in relational database tables, I used the following commands:

Input: `select * from Meals;`
Output:
```
+---------+----------------------+---------+--------------+------------+
| meal_id | meal_name            | cuisine | cooking_time | vegetarian |
+---------+----------------------+---------+--------------+------------+
|       1 | 
Spaghetti Bolognese | Italian |           45 |          0 |
|       2 | Vegetable Stir Fry   | Chinese |           20 |          1 |
|       3 | Chicken Curry        | Indian  |           50 |          0 |
|       4 | Mushroom Risotto     | Italian |           35 |          1 |
+---------+----------------------+---------+--------------+------------+
4 rows in set (0.00 sec)
```

Immediately, I notice that the formatting on the table for Spaghetti Bolognese is off. This does not seem to impact use of the table in any way, but I quickly realize that it was due to a syntax misinput on my part (user error). I Googled what command to use to edit the row in MySQL and used the `update` command to do so. After inputting `update Meals set meal_name = 'Spaghetti Bolognese', cuisine = 'Italian', cooking_time = 45, vegetarian = FALSE where meal_id = 1;` and running `select * from Meals;` again, I received the following output:
```
+---------+---------------------+---------+--------------+------------+
| meal_id | meal_name           | cuisine | cooking_time | vegetarian |
+---------+---------------------+---------+--------------+------------+
|       1 | Spaghetti Bolognese | Italian |           45 |          0 |
|       2 | Vegetable Stir Fry  | Chinese |           20 |          1 |
|       3 | Chicken Curry       | Indian  |           50 |          0 |
|       4 | Mushroom Risotto    | Italian |           35 |          1 |
+---------+---------------------+---------+--------------+------------+
4 rows in set (0.00 sec)
```

Input: `select * from Meals where vegetarian = TRUE;`
Output:
```
+---------+--------------------+---------+--------------+------------+
| meal_id | meal_name          | cuisine | cooking_time | vegetarian |
+---------+--------------------+---------+--------------+------------+
|       2 | Vegetable Stir Fry | Chinese |           20 |          1 |
|       4 | Mushroom Risotto   | Italian |           35 |          1 |
+---------+--------------------+---------+--------------+------------+
2 rows in set (0.00 sec)
```

Input: `select * from Meals order by cooking_time desc;`
Output: 
```
+---------+---------------------+---------+--------------+------------+
| meal_id | meal_name           | cuisine | cooking_time | vegetarian |
+---------+---------------------+---------+--------------+------------+
|       3 | Chicken Curry       | Indian  |           50 |          0 |
|       1 | Spaghetti Bolognese | Italian |           45 |          0 |
|       4 | Mushroom Risotto    | Italian |           35 |          1 |
|       2 | Vegetable Stir Fry  | Chinese |           20 |          1 |
+---------+---------------------+---------+--------------+------------+
4 rows in set (0.00 sec)
```

Input: `select * from Meals order by cooking_time asc;`
Output:
```
+---------+---------------------+---------+--------------+------------+
| meal_id | meal_name           | cuisine | cooking_time | vegetarian |
+---------+---------------------+---------+--------------+------------+
|       2 | Vegetable Stir Fry  | Chinese |           20 |          1 |
|       4 | Mushroom Risotto    | Italian |           35 |          1 |
|       1 | Spaghetti Bolognese | Italian |           45 |          0 |
|       3 | Chicken Curry       | Indian  |           50 |          0 |
+---------+---------------------+---------+--------------+------------+
4 rows in set (0.01 sec)
```

Input:
```
select Meals.meal_name as Meals,
	Ingredients.ingredient_name as Ingredients,
	Ingredients.quantity as Quantity
	from Meals
	join Ingredients on Meals.meal_id = Ingredients.meal_id;
```
Output:
```
+---------------------+-----------------+----------+
| Meals               | Ingredients     | Quantity |
+---------------------+-----------------+----------+
| Spaghetti Bolognese | Spaghetti       | 200g     |
| Spaghetti Bolognese | Ground Beef     | 250g     |
| Spaghetti Bolognese | Tomato Sauce    | 1 cup    |
| Vegetable Stir Fry  | Broccoli        | 100g     |
| Vegetable Stir Fry  | Carrots         | 50g      |
| Vegetable Stir Fry  | Soy Sauce       | 2T       |
| Chicken Curry       | Chicken Breast  | 300g     |
| Chicken Curry       | Curry Powder    | 2T       |
| Chicken Curry       | Coconut Milk    | 1 cup    |
| Mushroom Risotto    | Arborio Rice    | 1 cup    |
| Mushroom Risotto    | Mushrooms       | 1 cup    |
| Mushroom Risotto    | Parmesan Cheese | 1/2 cup  |
+---------------------+-----------------+----------+
12 rows in set (0.00 sec)
```

This references data in both tables, relating them to each other to produce an output that displays each ingredient and ingredient quantity with the meal recipe it is associated with (i.e. which foreign key meal_id it is associated with).

Input:
```
select ingredient_name as Ingredients,
	quantity as Quantity
	from Ingredients
	where meal_id = (select meal_id from Meals where meal_name = 'Chicken Curry');
```
Output:
```
+----------------+----------+
| Ingredients    | Quantity |
+----------------+----------+
| Chicken Breast | 300g     |
| Curry Powder   | 2T       |
| Coconut Milk   | 1 cup    |
+----------------+----------+
3 rows in set (0.01 sec)
```

This command queries which `meal_id` is associated with the `meal_name` Chicken Curry, and then returns all of the ingredients and quantities associated with that `meal_id` as a foreign key.

Input:
```
select cuisine, count(*) as meal_count
	from Meals
	group by cuisine;
```
Output:
```
+---------+------------+
| cuisine | meal_count |
+---------+------------+
| Italian |          2 |
| Chinese |          1 |
| Indian  |          1 |
+---------+------------+
3 rows in set (0.00 sec)
```

Input:
```
select meal_name, cooking_time
	from Meals
	where cooking_time <= 45
	order by cooking_time asc;
```
Output:
```
+---------------------+--------------+
| meal_name           | cooking_time |
+---------------------+--------------+
| Vegetable Stir Fry  |           20 |
| Mushroom Risotto    |           35 |
| Spaghetti Bolognese |           45 |
+---------------------+--------------+
3 rows in set (0.00 sec)
```

I then used the `\q` command to exit MySQL.

## Step 5: Creating a Bare Bones OPAC in HTML and PHP

After getting more experience creating, populating, and navigating relational databases in MySQL, I transitioned to working on the creation of a barebones OPAC to be hosted on my Apache-hosted website. The goal here to quote from the lecture notes, is "to acquire an intuition and understanding of how data from a relational database is retrieved and entered using LAMP technologies."

First, I used the following commands in MySQL to update the `books` table in `opacdb` so the `copyright` column uses the `DATE` data type, which will allow the search function (running in PHP) to filter by date.
```
mysql -u opacuser -p
mysql> use opacdb;
mysql> alter table books add publication_date date;
mysql> update books set publication_date = str_to_date(concat(copyright, '-01-01'), '%Y-%m-%d');
mysql> alter table books drop column copyright;
mysql> alter table books change publication_date copyright date not null;
```

After using `\q` to quit MySQL, I made a new HTML page, `mylibrary.html` in `/var/www/html`, with the following HTML code:
```
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>MySQL Server Example</title>
    </head>
<body>

    <h1>A Basic OPAC</h1>

    <p>In the form below, <b>optionally</b> enter text in the search field.
    Your search query will search by author, title, or publisher.
    Capitalization is usually not necessary on default case-insensitive MySQL collations.
    It's okay to enter partial information, like part of an author's, title's, or publisher's name.</p>

    <p>You can leave the search field empty and only enter dates.
    Regardless, both start and end dates are required for all searches.
    You can use the date fields to limit results, too.
    I added some extra records, which you can view to know what you can query:</p>

    <p><a href="opac.php">OPAC</a></p>

    <p>This is very much a toy, stripped down
    <a href="https://en.wikipedia.org/wiki/Online_public_access_catalog">OPAC</a>.
    The records are basic.
    Not only do they not conform to <a href="https://www.loc.gov/marc/">MARC</a>,
    they don't even conform to something as simple as <a href="https://www.dublincore.org/">Dublin Core</a>.</p>

    <p>I also don't provide options to select different fields, like author, title, or publisher fields.
    Instead the search field below searches key bibliographic fields (author, title, publisher) in our <b>books</b> table.</p>

    <p>The key idea is to get a sense of how an OPAC works, though.</p>

    <h2>My Basic Library OPAC</h2>

    <form method="post" action="search.php">
        <label for="search">Search Terms (optional):</label>
        <input type="text" name="search" id="search">
        
        <br>
        
        <label for="start_date">Start Date:</label>
        <input type="date" name="start_date" id="start_date" required>
        
        <br>
        
        <label for="end_date">End Date:</label>
        <input type="date" name="end_date" id="end_date" required>
        
        <br>
        
        <input type="submit" value="Search">
    </form>

</body>
</html>
```

This code creates a page with a search field that can receive user inputs to return contents from the `books` table in `opacdb`.

After saving, I created a PHP search script file called `search.php` in the same `/var/www/html` folder, with the following code:
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Search Results</title>
<style>
    table {
        border-collapse: collapse;
        width: 100%;
    }
    th, td {
        border: 1px solid black;
        padding: 8px;
        text-align: left;
    }
</style>
</head>
<body>

    <h1>Search Results</h1>

    <?php
    // Load MySQL credentials
    require_once '/var/www/login.php';

    // Enable MySQL error reporting
    mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);

    // Establish connection
    $conn = new mysqli($db_hostname, $db_username, $db_password, $db_database);
    if ($conn->connect_error) {
        die("Connection failed: " . $conn->connect_error);
    }

    if ($_SERVER["REQUEST_METHOD"] == "POST") {
        $search = trim($_POST['search']);
        $start_date = $_POST['start_date'];
        $end_date = $_POST['end_date'];

        // Prepared statement to prevent SQL injection
        $stmt = $conn->prepare("SELECT id, author, title, publisher, copyright FROM books 
                                WHERE (author LIKE ? OR title LIKE ? OR publisher LIKE ?) 
                                AND copyright BETWEEN ? AND ?");

        // Use wildcard search
        $search_param = "%$search%";
        $stmt->bind_param("sssss", $search_param, $search_param, $search_param, $start_date, $end_date);
        $stmt->execute();
        $result = $stmt->get_result();

        if ($result->num_rows > 0) {
            echo "<table>";
            echo "<tr><th>ID</th><th>Author</th><th>Title</th><th>Publisher</th><th>Copyright</th></tr>";

            while ($row = $result->fetch_assoc()) {
                echo "<tr>";
                echo "<td>" . htmlspecialchars($row["id"]) . "</td>";
                echo "<td>" . htmlspecialchars($row["author"]) . "</td>";
                echo "<td>" . htmlspecialchars($row["title"]) . "</td>";
                echo "<td>" . htmlspecialchars($row["publisher"]) . "</td>";
                echo "<td>" . htmlspecialchars($row["copyright"]) . "</td>";
                echo "</tr>";
            }

            echo "</table>";
        } else {
            echo "<p>No results found.</p>";
        }

        $stmt->close();
    }

    $conn->close();
    ?>

    <p><a href="mylibrary.html">Return to search page</a></p>

</body>
</html>
```

I also went back into MySQL to modify `opacdb` to insert a couple more books into the `books` table:
```
use opacdb;
insert into books
	(author, title, publisher, copyright) values
	('Emma Donoghue', 'Room', 'Little, Brown & Company', '2010-01-01'),
	('Zadie Smith', 'White Teeth', 'Hamish Hamilton', '2000-01-01');
```

I then used `select * from books;` to check that the books were present in the table; from this, I noticed a duplicate record for Zadie Smith's "White Teeth," implying that it was included in the original set of books as well as this above set of commands. I have left that duplicate in place for now. I then quit MySQL with the `\q` command.

## Step 6: Creating a Bare Bones Cataloguing Module: HTML Page and PHP Cataloguing Page

Another part of an ILS, separate from an OPAC, is a cataloguing system. Creating a cataloguing module that interfaces with the `opacdb` is very similar to creating the barebones OPAC, above.

First, I ran the following commands to navigate to the `html` directory, create a new directory called `cataloging`, and create a file called `index.html` in the `cataloging` directory:
```
cd /var/www/html
sudo mkdir cataloging
cd cataloging
sudo nano index.html
```

Then, in `/cataloging/index.html`, I inserted the following code:
```
<!DOCTYPE html>
<html>
<head>
    <title>Enter Records</title>
</head>
<body>
    <h1>OPAC Library Administration</h1>

    <p>This is the library administration page for entering records into the OPAC.</p>
    <p>Please do not use this page unless you are an authorized cataloger.</p>

    <form action="insert.php" method="post">
        <label for="author">Author:</label>
        <input type="text" name="author" id="author" required><br><br>

        <label for="title">Book Title:</label>
        <input type="text" name="title" id="title" required><br><br>

        <label for="publisher">Publisher:</label>
        <input type="text" name="publisher" id="publisher" required><br><br>

        <label for="copyright">Copyright:</label>
        <input type="date" name="copyright" id="copyright" required>

        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

Then, in the same folder, I created `insert.php`, which the above HTML code interfaces with. Using the command `sudo nano insert.php`, I created the PHP file and inserted the following code:
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cataloging: Data Entry</title>
</head>
<body>

<h1>Cataloging: Data Entry</h1>

<?php

// Load MySQL credentials
require_once '/var/www/login.php';

// Enable MySQL error reporting
mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);

// Establish connection
$conn = new mysqli($db_hostname, $db_username, $db_password, $db_database);
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

if ($_SERVER["REQUEST_METHOD"] === "POST") {
    $author = trim($_POST["author"] ?? "");
    $title = trim($_POST["title"] ?? "");
    $publisher = trim($_POST["publisher"] ?? "");
    $copyright = $_POST["copyright"] ?? "";

    if ($author === "" || $title === "" || $publisher === "" || $copyright === "") {
        echo "All fields are required.";
    } elseif (!preg_match('/^\d{4}-\d{2}-\d{2}$/', $copyright)) {
        echo "Copyright date must use YYYY-MM-DD format.";
    } else {
        // Prepare and bind SQL statement
        $stmt = $conn->prepare("INSERT INTO books (author, title, publisher, copyright) VALUES (?, ?, ?, ?)");
        $stmt->bind_param("ssss", $author, $title, $publisher, $copyright);

        if ($stmt->execute() === TRUE) {
            echo "New record created successfully";
        } else {
            echo "Error: " . $stmt->error;
        }
        $stmt->close();
    }
} else {
    echo "Please submit records using the cataloging form.";
}

// Close connection
$conn->close();
?>

<p><a href='index.html'>Return to Cataloging Page</a></p>
<p><a href='../mylibrary.html'>Return to Library Home Page</a></p>
</body>
</html>
```

I checked that this was working as intended by visiting the `/cataloging/index.html` page on my Apache-hosted website. Upon visiting the page, it displayed correctly.

## Step 7: Establishing Security Protocols for the Cataloguing Module

Since the HTML and PHP files for the cataloguing module allow direct access to and modification of the MySQL database through the web, we need to limit access using security protocols.

To begin with, I created an authentication file in the `etc/apache2` directory and set the username to `libcat` using the `sudo htpasswd -c /etc/apache2/.htpasswd libcat`. This prompted me to input a password.

I then opened the `apache2.conf` file in nano and added the following code into the file below the existing `<Directory /var/www/>` block of code.
```
<Directory /var/www/html/cataloging/>
  Options Indexes FollowSymLinks
  AllowOverride AuthConfig
  Require all granted
</Directory>
```

After saving and then navigating back to the `cataloging` directory with `cd /var/www/html/cataloging`, I used the command `sudo nano .htaccess` to create a new file by that name and inserted the following code into it:
```
AuthType Basic
AuthName "Authorization Required"
AuthUserFile /etc/apache2/.htpasswd
Require valid-user
```

This file will apply to web requests for everything in the `/var/www/html/cataloging` directory, prompting for a username and password to access.

Using the command `sudo apachectl configtest` I confirmed that syntax was OK.

Then, I restarted Apache and checked its status with `sudo systemctl restart apache2` and `sudo systemctl status apache2`. This confirmed that Apache is enabled, active, and has no errors.

In order to allow Apache to write data (including uploading directories), I went and changed ownership over `/var/www/html` to `www-data` (the user which allows Apache to write data) with the command `sudo chown :www-data /var/www/html`.

Then, using the command `sudo find /var/www/html -type d -exec chmod g+s {} +`, I set it so that any new files and directories created within `/var/www/html` will inherit the group ownership of `www-data`. 

## Issues Encountered

All issues noted above, including syntax errors in MySQL, etc., were due to user error. By using resources available online on CLI commands, I was able to modify mistyped entries in MySQL.

## Reflection

This was comparatively more involved work than previous modules, but once it all began to come together it made a lot of sense! I learned how to apply the skills learned in previous modules (using HTML, PHP, and MySQL to create a simple OPAC that can be used in-browser), and began to really see how this all comes together into the kind of work that is done in libraries (namely work on ILSes). With the OPAC and cataloguing module made and accessible in browser, doing data entry to add many many more records to our `opacdb` `books` table will be much easier. I look forward to using these tools to build a larger library website with an OPAC and library catalog!

## Conclusion

MySQL has been updated, and webpages for a barebones OPAC and a cataloguing module have been created. The cataloguing module is set to require a username and password to access for security reasons, since it gives direct access to modifying the files in the directory from the browser. All of this is displaying correctly, and the password prompt is functioning as intended.
