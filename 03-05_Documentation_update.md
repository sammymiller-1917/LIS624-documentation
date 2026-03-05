## 2026-03-05 - Major Documentation Update

### Goal

Continuing to learn the CLI and additional systems librarianship-relevant skills in the LIS 624 class. Mass updating documentation on work done in the last month, in order to set a good foundation for future regular documentation updates.

### Context

No documentation was written for work done over the course of the month of February, after a Readme update on 2026-02-06. In order to set a good foundation for future regular documentation updates, one large update with cover all work from the month of February. This includes work related to the following skills:
- Using ` grep` for line searches within a fil
- Using `apt` to manage, install, update, and upgrade software on the VM
- Searching a library database with `yaz-client`
- Hosting a website on Apache

All of the above was done on the spring-2026 VM, which runs Ubuntu Linux.

### Steps

#### Using grep

The grep command can be used for searching for a given string within a file. grep returns the entire line in which a provided string appears, but with further modifiers a user can have grep return an inverse match, the number of lines a string appears on, etc.

Modifiers for grep include:
- `-v` : inverse search
- `-i` : turn off case sensitivity
- `-c` : returns a count of matching lines
- `|` : Boolean OR search
- `-w` : whole word matching
- `-A` NUM : tells grep to print the matching line and the NUM lines after the matching line
- `-B` NUM : tells grep to print the matching line and the NUM lines before the matching line
- `-C` NUM : tells grep to print the matching line and the NUM lines before and after it
- `-m` NUM : tells grep to stop a search after NUM number of hits
- `-n` : return line numbers

In addition, we can further run commands like `sort`, `uniq`, etc. which will further refine grep searches and make them more manageable. `sort` orders outputs alphabetically, while `uniq` deduplicates results.

To get practice using grep and related commands, I downloaded the BIB file of a search for "Islamic History" from Web of Science, and uploaded the file (savedrecs.bib) to the VM. Using the `grep` command and many of the above modifiers, I performed searches for certain keywords, article titles, publishing languages, etc. within the metadata in savedrecs.bib.

#### Using apt and sudo

The `apt` command is used for managing the software installed on the VM. Using the package manager to actually install a piece of software, however, also requires the use of the `sudo` command. The `sudo` command by default executes a command as the superuser account.

These two commands are often used together. The following are commands used for managing software using both `apt` and `sudo`:
- `sudo apt update` : updates the list of installed software and checks that each software is up to date to the most up-to-date version available.
- `sudo apt upgrade` : upgrades all installed software for whom updates are available (check with sudo apt update first).
- `sudo apt install` : installs a specified package.
- `sudo apt remove` : removes a specified package (add the `--purge` modifier to remove its system config files).
- `sudo apt autoremove` : uninstalls software dependencies.
- `sudo apt clean` : cleans cached package files, should be run after an installation to free disk space.

The `apt` command can also be used on its own to search for and show information on a specific package prior to installation (using the `apt search` and `apt show` commands).

I used `apt` and `sudo` to install `yaz-client`, as well as other pieces of software necessary for further coursework. I first used `apt search` to find `yaz-client`, `apt show` to learn about the program, and then installed it with `sudo apt install`. Since then, I have also used `sudo apt update` and `sudo apt upgrade` every time I have opened the VM, in order to keep my software up-to-date.

#### Using yaz

The `yaz-client` command is able to connect to and search through a library database. It is not a line search system like `grep`, but instead performs 739.50/SRU searches, which has some overlap with Boolean searches (although `yaz` formats queries differently).

To use `yaz-client` to search a library database, do the following:
- run `yaz-client`
- run the `open` command to connect to a specific library catalog (for instance, to connect to UKY's library catalog, run `open saalck-uky.alma.exlibrisgroup.com:1921/01SAA_UKY` )
- while `yaz-client` is connected to the library database, use the `find` command to perform searches using queries formatted with Prefix Query Format (PQF), putting Boolean operators at the beginning of the query.

Examples of operators and modifiers  usable with the `yaz-client` include:
- `@and` : Boolean AND
- `@or` : Boolean OR
- `@not` : Boolean NOT
- `@attr 1=4` : search for the immediately following term in the Title field
- `@attr 1=21` : search for the immediately following term in the subject heading field
- `@attr 1=1` : search for the immediately following term in the personal name field
- `@attr 1=1016` : search for the immediately following term in ANY field
- `-m` : used as a modifier on the initial yaz-client command. Appends bibliographic records to a specified file.

The command `yaz-marcdump` can be used to convert a MARC file downloaded with yaz to a different filetype (for example JSON). Another relevant command is `jq`, which is used to process JSON files for readability, extract certain fields into a separate file, etc.

After installing yaz, I used the `yaz-client` command to perform searches in the UKY Library Catalog, downloaded a set of bibliographic records, and used `jq` to reformat the records into JSON and reformat them.

#### Installing the Apache Web Server

Installing and configuring Apache as a simple web server for hosting my own website is the most significant change that I have made to the VM since the establishment of the Github repo.

Steps taken:
- Installed Apache using `sudo apt install apache2`
- Used `systemctl status apache2` to check that Apache was enabled and running
- Installed w3m using `sudo apt install w3m`
- Checked the webpage in both w3m in the command line interface and by visiting the VM's IP address in a graphical browser (namely Firefox)
- Made a new directory using `cd /var/www/html/`
- Renamed the existing `index.html` file `index.original.html` using the `sudo  mv command`
- Created a new `index.html` command using `sudo nano index.html`
- Edited the HTML of the webpage in nano (refer to `index.html` in the Github repo for the content of the code)

### Results

Installed multiple useful programs onto the VM, including `grep`, `yaz`, and Apache. Got practice navigating metadata and Library Catalogs using `grep` and `yaz-client`. Set up the Apache web server and created my first self-hosted webpage on Apache. As part of that process, created a new directory, `/var/www/html`.

### Verification

To verify the Apache web server is working as intended, visit http://34.170.249.97/

### Notes

This documentation was not written in standard documentation notation, owing to being retroactive and covering multiple installations, softwares, etc. in one document. Future documentation will be in standard documentation formatting.
