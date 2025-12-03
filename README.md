{"id":"18403","variant":"email","title":"README - FreeRADIUS on Ubuntu 22.04"}
# FreeRADIUS on Ubuntu 22.04

<p align="center">
  <strong>Step-by-step guide to install and test <em>FreeRADIUS</em> on <em>Ubuntu 22.04</em></strong>
</p>

<p align="center">
  <a href="#"><img src="https://img.shields.io/badge/Ubuntu-22.04-orange" alt="Ubuntu 22.04"></a>
  <a href="#"><img src="https://img.shields.io/badge/FreeRADIUS-Compatible-blue" alt="FreeRADIUS"></a>
  <a href="#"><img src="https://img.shields.io/badge/License-MIT-green" alt="License"></a>
</p>

---

## About

This repository documents how to install and configure **FreeRADIUS** on **Ubuntu 22.04**.  
It's intended for hotspot / ISP deployments (for example: **MikroTik + FreeRADIUS + external captive portal**).

---

## Features / What this guide covers

- Installing FreeRADIUS and utilities  
- Starting and enabling the service  
- Running FreeRADIUS in debug mode for troubleshooting  
- Creating a test user and testing authentication locally  
- Example MikroTik NAS client configuration  
- Opening firewall ports and checking server listening status  
- Basic log locations and commands  
- Optional: enabling SQL (MariaDB) backend and the freeradius-mysql module

---

## Quick Start — Commands

> Run the following commands on your Ubuntu 22.04 server (use `sudo` where required).

### 1. Update the system
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install FreeRADIUS and utilities
```bash
sudo apt install freeradius freeradius-utils -y
```

Check the installed version:
```bash
freeradius -v
```

### 3. Start and enable FreeRADIUS service
```bash
sudo systemctl start freeradius
sudo systemctl enable freeradius
sudo systemctl status freeradius
```
You should see `active (running)` if the service started successfully.

### 4. Test FreeRADIUS in debug mode
Run the server in the foreground with verbose logging to observe errors and flows:
```bash
sudo freeradius -X
```
If it starts without errors, the installation is OK.

### 5. Create a test user
Edit the users file:
```bash
sudo nano /etc/freeradius/3.0/users
```
Add this test account at the end:
```
testuser Cleartext-Password := "testpass"
```
Save and restart the service:
```bash
sudo systemctl restart freeradius
```

### 6. Test authentication locally
Use `radtest` to check RADIUS authentication:
```bash
radtest testuser testpass 127.0.0.1 0 testing123
```
Successful authentication will return `Access-Accept`.

### 7. Configure a NAS client (MikroTik example)
Edit RADIUS clients:
```bash
sudo nano /etc/freeradius/3.0/clients.conf
```
Add a client block for your MikroTik (example):
```
client mikrotik {
    ipaddr = 192.168.88.1
    secret = mysharedsecret
    require_message_authenticator = no
    nas_type = other
}
```
Restart FreeRADIUS:
```bash
sudo systemctl restart freeradius
```

### 8. Open firewall ports
FreeRADIUS uses UDP ports **1812** (authentication) and **1813** (accounting):
```bash
sudo ufw allow 1812/udp
sudo ufw allow 1813/udp
sudo ufw reload
```

### 9. Verify the server is listening
```bash
sudo netstat -anu | grep radius
```
(or `ss -anu | grep 1812`)

### 10. Logs & debugging
- Debug output (foreground):
```bash
sudo freeradius -X
```
- Standard log file:
```bash
sudo tail -f /var/log/freeradius/radius.log
```

---

## Optional: MySQL / MariaDB backend (SQL)
If you want FreeRADIUS to store/lookup users in SQL:

1. Install MariaDB:
```bash
sudo apt install mariadb-server -y
```

2. Install the FreeRADIUS MySQL module:
```bash
sudo apt install freeradius-mysql -y
```

3. Configure the SQL module (`/etc/freeradius/3.0/mods-enabled/sql`) and import schema into your database. (See `mods-available/sql` for example configs and SQL schema files to import.)

---

## Next steps (suggested documentation to add)
- CHAP authentication configuration (challenge/response) for hotspots  
- MikroTik hotspot integration details (how to point hotspot to RADIUS + secrets + NAS configuration)  
- External captive portal (Laravel) flow: how to map payment → RADIUS accept (CoA or dynamic user entry)  
- Using SQL (users and accounting) with summaries and sample SQL configs  
- Example clients configuration files (add `examples/` folder to repo)

---

## Contributing

Contributions and improvements are welcome. Please:
1. Fork the repository  
2. Create a branch (e.g. `docs/chap-mikrotik`)  
3. Submit a PR with clear changes and reasons

---

## Security Vulnerabilities

If you discover a security vulnerability, please open an issue immediately and mark it as a security concern. Include steps to reproduce and any logs if possible. Sensitive security issues can alternatively be disclosed through a private channel if you prefer.

---

## License

This project is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).


"# freeradius-ubuntu-22.04" 
