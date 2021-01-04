# Zabbix Server 5.2 install with Apache and MariaDB on Ubuntu 20.04

### 1. Update the system

```bash
sudo apt update

sudo apt upgrade
```

------

### 2. Install mariaDB

```bash
sudo apt -y install mariadb-server

sudo systemctl enable --now mariadb

sudo systemctl status mariadb
```

------

### 3. Secure mariaDB

```bash
mysql_secure_installation
```

```
Enter current password for root (enter for none): Press the Enter
Set root password? [Y/n]: Y
New password: <Enter root DB password>
Re-enter new password: <Repeat root DB password>
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]: Y
Reload privilege tables now? [Y/n]: Y
```

------

### 4. Create initial database

```bash
mysql -uroot -p
```

```sql
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

------

### 5. Install Zabbix repository

```bash
wget https://repo.zabbix.com/zabbix/5.2/ubuntu/pool/main/z/zabbix-release/zabbix-release_5.2-1+ubuntu20.04_all.deb

sudo dpkg -i zabbix-release_5.2-1+ubuntu20.04_all.deb

sudo apt update
```

------

### 6. Install Zabbix Server, Zabbix Agent and Zabbix frontend

```bash
sudo apt install zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-agent
```

------

### 7. import initial schema and data to the database.

```bash
sudo zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

------

### 8. Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf

```
DBPassword=password
```
------

### 9. Start Zabbix server and agent processes

```bash
# restarts the zabbix-server, zabbix-agent, apache and php service.
sudo systemctl restart zabbix-server zabbix-agent apache2

# enables the zabbix-server, zabbix-agent, apache and php service to start automatically after a reboot.
sudo systemctl enable zabbix-server zabbix-agent apache2

# gets the status of zabbix-server, zabbix-agent, apache and php service.
sudo systemctl status zabbix-server zabbix-agent apache2
```

------

### 10. Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name/zabbix

