# Zabbix Server install with Nginx and PostgreSQL on Debian 10

### 1. Update the system

```bash
sudo apt update

sudo apt upgrade
```

------


### 2. Install postgresql

```bash
sudo apt install postgresql
```

------

### 3. Add a password to the Postgres user.

```bash
sudo passwd postgres
```

------

### 4. Add the postgres user to the sudo group

```bash
sudo usermod -aG sudo postgres
```

------

### 5. Install Zabbix repository

```bash
wget https://repo.zabbix.com/zabbix/5.2/debian/pool/main/z/zabbix-release/zabbix-release_5.2-1+debian10_all.deb

sudo dpkg -i zabbix-release_5.2-1+debian10_all.deb

sudo apt update
```

------

### 6. Install Zabbix Server, Zabbix Agent and Zabbix frontend

```bash
sudo apt install zabbix-server-pgsql zabbix-frontend-php php7.3-pgsql zabbix-apache-conf zabbix-agent
```

------

### 7. Create initial database

```bash
# Switch to the postgres user.
su - postgres

# Create a database user called zabbix
sudo -u postgres createuser --pwprompt zabbix

# Create a database with UTF8 called Zabbix and give ownership to the zabbix user created before
sudo -u postgres createdb -O zabbix -E Unicode -T template0 zabbix
```

------

### 8. import initial schema and data to the database.

```bash
sudo zcat /usr/share/doc/zabbix-server-pgsql*/create.sql.gz | sudo -u zabbix psql zabbix
```

------

### 9. Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf

Add the password you created for the [Zabbix database user](#7-create-initial-database).

```
DBPassword=password
```

------

### 10. Configure PHP for Zabbix frontend

Edit file /etc/zabbix/apache.conf, uncomment and set the right timezone for you.

```
# php_value date.timezone Europe/Riga
```

------

### 11. Start Zabbix server and agent processes

```bash
sudo systemctl restart zabbix-server zabbix-agent apache2

sudo systemctl enable zabbix-server zabbix-agent apache2

sudo systemctl status zabbix-server zabbix-agent apache2
```

------

### 12. Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name/zabbix

