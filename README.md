
<p align="center">
    <img src="https://image.pngaaa.com/954/1486954-middle.png" width="320" alt="FreeRADIUS Logo">
</p>

<p align="center">
    <img src="https://img.shields.io/badge/Ubuntu-22.04-orange" />
    <img src="https://img.shields.io/badge/FreeRADIUS-3.x-blue" />
    <img src="https://img.shields.io/badge/Backend-SQL-green" />
    <img src="https://img.shields.io/badge/License-MIT-brightgreen" />
</p>


## How To Install FreeRadius on Ubuntu 22.04

> How to install a RADIUS Server with a SQL backend.


### Intro
FreeRadius is an Ubuntu software that acts as a RADIUS server your router can use to give you a 802.11x network. There are various reasons you may want to do this, but today I'll show you how to get it installed.

Documentation last verified on 1/12/25 using Ubuntu Server 22.04 LTS. All commands function as expected.

Looking for IT consulting? Hit us up! https://mathewekuwam.dev/it-consulting.html


### Getting Started
The first thing you'll want to do is ensure your system is up to date. You can do this by running the following commands to update the available packages on your system.
```bash
sudo apt update
sudo apt upgrade -y
```
Next, there are some packages you need to install that you might not have already. A few notable ones are php, apache2, FreeRadius, and mariadb. 
```bash
sudo apt install php apache2 php8.1-fpm freeradius libapache2-mod-php mariadb-server freeradius-mysql freeradius-utils php-{gd,common,mail,mail-mime,mysql,pear,db,mbstring,xml,curl} -y
```
Then, you need to enable and start apache2 as well as enable FreeRadius. You can do this inside of one command listed below.
```bash
sudo systemctl enable --now apache2 && sudo systemctl enable freeradius
```
Once you have apache2 enabled, you can setup the sql server (in this case mariadb server).
```bash
sudo mysql_secure_installation
```
Here's what I answered during the installation, depending on your security needs, you may want to set a root password, but in my case it doesn't matter to me. I pasted the questions as well as my answers to them (right after the questions)
```bash
Enter current password for root (enter for none): enter

Switch to unix_socket authentication [Y/n] n

Change the root password? [Y/n] n

Remove anonymous users? [Y/n] y

Disallow root login remotely? [Y/n] y

Remove test database and access to it? [Y/n] y

Reload privilege tables now? [Y/n] y
 
```
### Setting up mariadb
Great, now that you have setup the server, we can start to configure the database. Depending on how you set it up, you may have to enter your sql root password on this first command, but since I clicked enter through the prompt, I will just type in sudo mysql. If you entered a root password, type in the command below.
```bash
sudo mysql -u root -p
```
Now that you're logged in, you need to do a few things. 

Create the databsae
Create a user for FreeRadius
Give the user permissions
We can create the database by running this “CREATE DATABASE” command:
```bash
CREATE DATABASE radius;
```
Next we need to create the user account. I would recommend changing PASSWORD to a secure password.
```bash
CREATE USER 'radius'@'localhost' IDENTIFIED by 'PASSWORD';
```
To grant the privileges, run this command:
```bash
GRANT ALL PRIVILEGES ON radius.* TO 'radius'@'localhost';
```
Finally, run these commands to reload the privileges in the sql database and to quit the session.
```bash
FLUSH PRIVILEGES;
quit;
```

To finish setting up the database, we need to connect it to FreeRadius. To tell FreeRadius we will use sql for our logins, run these commands below one at a time. You cannot paste all these in at once.
```bash
sudo su -

mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql

exit

sudo ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/

```

### Telling FreeRadius the SQL Login
Now that FreeRadius and the SQL server are connected, we need to provide FreeRadius with the login to the SQL server, as well as change a few settings. 

```bash
sudo nano /etc/freeradius/3.0/mods-enabled/sql
```
There are a few things you need to change before entering in the login. Below are the things that need changed. Scroll through and make the necessary changes.

driver = "rlm_sql_null" needs to be driver = "rlm_sql_${dialect}"

dialect = "sqlite" needs to be dialect = "mysql"

read_clients = yes needs to be uncommented (No # in front of that line)

client_table = “nas” needs to be uncommented (No # in front of that line)

One last thing, find the section that looks like this:


Default TLS settings for the SQL connection in FreeRadius
And comment out all the TLS settings to make it look like this:


Corrected TLS settings for the SQL connection in FreeRadius
Now you can enter in the login. Find the section that looks like this:


Default values filled in to “Connection Info” on FreeRadius
And uncomment the lines that say server, port, login, and password. (Note: Uncommenting means removing the #). Enter in your radius user password for your SQL database in the “” where it says “radpass”.

Here is what it should look like:


Correctly filled in “Connection Info” for the SQL database on FreeRadius
To save your changes, on your keyboard, click ctrl x, y, followed by enter.

### Finishing FreeRadius config
```bash
sudo chgrp -h freerad /etc/freeradius/3.0/mods-available/sql
sudo chown -R freerad:freerad /etc/freeradius/3.0/mods-enabled/sql
sudo systemctl restart freeradius
```
### Optional: Phpmyadmin Installation

This is completely optional, however I recommend installing phpmyadmin to monitor your FreeRadius database a bit easier (since it has a web interface). This is part of the reason I had you install apache2 earlier.

sudo apt install phpmyadmin
When it comes up with the automated installer, select apache2, enter in your sql settings, and boom, you're done. 

Navigate to the IP of your FreeRadius server in a web browser and go to /phpmyadmin.

 

### Dynamic VLAN assignment
If you're like me and you have UniFi network hardware, there'll be an extra step you want to take. 

Run these commands SEPARATELY and find the SECOND OCCURRENCE of “use_tunneled_reply” in the file that'll open..
```bash
sudo nano /etc/freeradius/3.0/mods-enabled/eap
```
In the second occurrence of “use_tunneled_reply", under the “peap” section, change NO to YES. This will allow UniFi to dynamically assign users's vlans based on their login stored in the SQL database.

### Good to know
One last thing, every time you make a change in your FreeRadius settings (that includes every time you add or modify a user) you need to reload your settings. You can do that by running this command below:
```bash
sudo service freeradius reload
```
### Connecting to the RADIUS server
Finally, to connect to the server, you will need to enter in your router's PRIVATE IP ADDRESS as a “NAS” in the nas table in your database. In that row, you'll also need to create a “secret” that your router will use to connect to the radius server. This should be pretty straight forward so I'm not going to go into too much detail on here, but if you have questions, like always, feel free to reach out on my website,mathewekuwam.dev.

### Bonus: Migrating RADIUS servers
If you want to configure a backup server or just migrate your existing server, you'll notice that you have to “re trust” the certificate for your Freeradius server. Not anymore! If you copy over the 3 following ssl certs, it won't ask you to re trust it anymore!

Here are the three files you need to transfer over. Transfer them AFTER you setup your new server, then remove the files from the new server, copy from old, and restart Freeradius.
```bash
/etc/ssl/private/ssl-cert-snakeoil.key
/etc/ssl/certs/ssl-cert-snakeoil.pem
/etc/ssl/certs/ca-certificates.crt
```
### Note: Unsure if the last one matters or not, haven't tried it. Doesn't seem to “hurt” anything if it is copied over.
