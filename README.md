<p align="center">
    <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/0/0c/FreeRADIUS_Logo.png/600px-FreeRADIUS_Logo.png" width="320" alt="FreeRADIUS Logo">
</p>

<p align="center">
    <img src="https://img.shields.io/badge/Ubuntu-22.04-orange" />
    <img src="https://img.shields.io/badge/FreeRADIUS-3.x-blue" />
    <img src="https://img.shields.io/badge/Backend-SQL-green" />
    <img src="https://img.shields.io/badge/License-MIT-brightgreen" />
</p>

# How To Install FreeRadius on Ubuntu 22.04  
### How to install a RADIUS Server with a SQL backend.

---

## 📘 Intro

FreeRadius is an Ubuntu software that acts as a RADIUS server your router can use to give you a 802.11x network. There are various reasons you may want to do this, but today I'll show you how to get it installed.

Documentation last verified on **1/12/25** using **Ubuntu Server 22.04 LTS**.  
All commands function as expected.

Looking for IT consulting? Hit us up!  
👉 **https://mathewekuwam.dev/it-consulting.html**

---

## 🚀 Getting Started

The first thing you'll want to do is ensure your system is up to date.

### Update system packages  
```bash
sudo apt update
sudo apt upgrade -y
```
Install required packages
A few notable ones are PHP, Apache2, FreeRADIUS, and MariaDB.

```bash
sudo apt install php apache2 php8.1-fpm freeradius libapache2-mod-php mariadb-server freeradius-mysql freeradius-utils php-{gd,common,mail,mail-mime,mysql,pear,db,mbstring,xml,curl} -y
```
Enable Apache2 and FreeRADIUS
```bash
sudo systemctl enable --now apache2 && sudo systemctl enable freeradius
Secure MariaDB
```

```bash
sudo mysql_secure_installation
```
Answers used during setup
```bash
Enter current password for root (enter for none): enter
Switch to unix_socket authentication [Y/n] n
Change the root password? [Y/n] n
Remove anonymous users? [Y/n] y
Disallow root login remotely? [Y/n] y
Remove test database and access to it? [Y/n] y
Reload privilege tables now? [Y/n] y
```

🗄️ Setting up MariaDB
Login to MariaDB
If you set a root password, enter it when prompted:

```bash
sudo mysql -u root -p
```
Inside MariaDB:

1. Create the database
```bash
CREATE DATABASE radius;
```
2. Create a user (replace PASSWORD)
```bash
CREATE USER 'radius'@'localhost' IDENTIFIED BY 'PASSWORD';
```
3. Grant permissions
```bash
GRANT ALL PRIVILEGES ON radius.* TO 'radius'@'localhost';
```
4. Reload privileges and exit
```bash
FLUSH PRIVILEGES;
quit;
```
🔗 Connecting MariaDB to FreeRADIUS
Run these commands one at a time:

```bash
sudo su -
mysql -u root -p radius < /etc/freeradius/3.0/mods-config/sql/main/mysql/schema.sql
exit
sudo ln -s /etc/freeradius/3.0/mods-available/sql /etc/freeradius/3.0/mods-enabled/
```
🔧 Telling FreeRadius the SQL Login
Open the SQL module configuration:

```bash
sudo nano /etc/freeradius/3.0/mods-enabled/sql
```
Scroll and make the following changes:

Required modifications
Original    Change To
driver = "rlm_sql_null" driver = "rlm_sql_${dialect}"
dialect = "sqlite"  dialect = "mysql"
#read_clients = yes read_clients = yes
#client_table = "nas"   client_table = "nas"

Disable TLS
Find the TLS block and comment out everything inside it.

Add SQL login credentials
Locate the Connection Info section and uncomment:

```bash
server = "localhost"
port = 3306
login = "radius"
password = "radpass"
Replace "radpass" with your SQL user password.
```

Save your changes: CTRL + X → Y → ENTER

🏁 Finishing FreeRadius Configuration
```bash
sudo chgrp -h freerad /etc/freeradius/3.0/mods-available/sql
sudo chown -R freerad:freerad /etc/freeradius/3.0/mods-enabled/sql
sudo systemctl restart freeradius
```

🌐 Optional: phpMyAdmin Installation
This helps monitor your RADIUS SQL database through a web interface.

```bash
sudo apt install phpmyadmin
```

Select apache2, enter your SQL details, then access via:

```bash
http://YOUR_SERVER_IP/phpmyadmin
```

🔀 Dynamic VLAN Assignment (UniFi & Others)
Edit the EAP module:

```bash
sudo nano /etc/freeradius/3.0/mods-enabled/eap
```
Find the second occurrence of:

```bash
use_tunneled_reply = no
```
Under the peap section, change it to:

```bash
use_tunneled_reply = yes
```

This enables UniFi to dynamically assign VLANs based on the user’s SQL profile.

🔄 Good to Know
Every time you modify FreeRADIUS settings—including adding or editing users—you must reload:

```bash
sudo service freeradius reload
```
📡 Connecting Devices to the RADIUS Server
To connect to the server, add your router’s private IP address as a NAS entry in the nas table in your database.

You must also define a shared secret that both your router and FreeRADIUS will use.

If you have questions, reach out:
👉 mathewekuwam.dev

🎁 Bonus: Migrating RADIUS Servers
To avoid retrusting certificates, copy these three SSL files from the old server to the new one (after setup):

```bash
/etc/ssl/private/ssl-cert-snakeoil.key
/etc/ssl/certs/ssl-cert-snakeoil.pem
/etc/ssl/certs/ca-certificates.crt
```

Note: unsure if the last certificate is mandatory, but it causes no issues if transferred.
