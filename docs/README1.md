<p align="center"> <a href="https://freeradius.org" target="_blank"> <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/0c/FreeRADIUS_Logo.png/600px-FreeRADIUS_Logo.png" width="320" alt="FreeRADIUS Logo"> </a> </p> <p align="center"> <a href="#"><img src="https://img.shields.io/badge/Ubuntu-22.04-orange" alt="Ubuntu 22.04"></a> </p>
How To Install FreeRADIUS on Ubuntu 22.04

FreeRADIUS is an Ubuntu-based RADIUS server that provides authentication, authorization, and accounting (AAA) for 802.1X networks. This guide covers installing FreeRADIUS with a SQL backend (MariaDB) and connecting it to your network devices.

Documentation last verified: 1/12/25 using Ubuntu Server 22.04 LTS. All commands tested successfully.

Table of Contents

Getting Started

Setting up MariaDB

Connecting FreeRADIUS to SQL

Configuring FreeRADIUS SQL Login

Optional: PhpMyAdmin

Dynamic VLAN Assignment

Reloading FreeRADIUS

Connecting Network Devices

Migrating FreeRADIUS Servers

License

Getting Started

First, ensure your system is updated:

sudo apt update
sudo apt upgrade -y


Install the required packages including PHP, Apache2, FreeRADIUS, and MariaDB:

sudo apt install php apache2 php8.1-fpm freeradius libapache2-mod-php mariadb-server freeradius-mysql freeradius-utils php-{gd,common,mail,mail-mime,mysql,pear,db,mbstring,xml,curl} -y


Enable and start Apache2 and FreeRADIUS:

sudo systemctl enable --now apache2 && sudo systemctl enable freeradius


Secure the MariaDB installation:

sudo mysql_secure_installation


Sample answers for installation prompts:

Enter current password for root (enter for none): enter
Switch to unix_socket authentication [Y/n] n
Change the root password? [Y/n] n
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y

Setting up MariaDB

Log in to MariaDB as root:

sudo mysql -u root -p


Create the database for FreeRADIUS:

CREATE DATABASE radius;


Create a dedicated user (replace PASSWORD with a secure password):

CREATE USER 'radius'@'localhost' IDENTIFIED BY 'PASSWORD';


Grant privileges and reload:

GRANT ALL PRIVILEGES ON radius.* TO 'radius'@'localhost';
FLUSH PRIVILEGES;
quit;

Connecting FreeRADIUS to SQL

Import the FreeRADIUS schema into MariaDB:

sudo su -
mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql
exit


Enable the SQL module for FreeRADIUS:

sudo ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/

Configuring FreeRADIUS SQL Login

Edit the SQL module configuration:

sudo nano /etc/freeradius/3.0/mods-enabled/sql


Make the following changes:

driver = "rlm_sql_null" → driver = "rlm_sql_${dialect}"

dialect = "sqlite" → dialect = "mysql"

Uncomment read_clients = yes

Uncomment client_table = "nas"

Comment out TLS settings

Enter your SQL connection credentials:

server = "localhost"
port = 3306
login = "radius"
password = "radpass"


Save changes: Ctrl+X → Y → Enter

Fix permissions and restart FreeRADIUS:

sudo chgrp -h freerad /etc/freeradius/3.0/mods-available/sql
sudo chown -R freerad:freerad /etc/freeradius/3.0/mods-enabled/sql
sudo systemctl restart freeradius

Optional: PhpMyAdmin Installation

Install PhpMyAdmin to monitor the database through a web interface:

sudo apt install phpmyadmin


Select Apache2 during installation

Enter SQL credentials when prompted

Access PhpMyAdmin via http://<server-ip>/phpmyadmin

Dynamic VLAN Assignment

If using UniFi hardware, edit EAP module:

sudo nano /etc/freeradius/3.0/mods-enabled/eap


Find the second occurrence of use_tunneled_reply under peap

Change NO → YES

This allows dynamic VLAN assignment based on user login.

Reloading FreeRADIUS

Every time you add or modify a user or change settings:

sudo service freeradius reload

Connecting Network Devices

Add your router’s PRIVATE IP as a NAS in the nas table

Create a shared secret for authentication

Migrating FreeRADIUS Servers

Copy the following SSL certificates from the old server to the new one after setup:

/etc/ssl/private/ssl-cert-snakeoil.key
/etc/ssl/certs/ssl-cert-snakeoil.pem
/etc/ssl/certs/ca-certificates.crt


This avoids re-trusting certificates on the new server.

License

This project is open-sourced software licensed under the MIT License
.
