# Zabbix Server install with Nginx and PostgreSQL on Debian 10

### 1. Update the system

```bash
sudo apt update

sudo apt upgrade
```

------

### 2. Install nginx

```bash
sudo apt install nginx
```

------

### 3. Install postgresql

```bash
sudo apt install postgresql
```

------

### 4. Add a password to the Postgres user.

```bash
sudo passwd postgres
```

------

### 5. Add the postgres user to the sudo group

```bash
sudo usermod -aG sudo postgres
```

------

### 6. Install Zabbix repository

```bash
wget https://repo.zabbix.com/zabbix/5.2/debian/pool/main/z/zabbix-release/zabbix-release_5.2-1+debian10_all.deb

sudo dpkg -i zabbix-release_5.2-1+debian10_all.deb

sudo apt update
```

------

### 7. Install Zabbix Server, Zabbix Agent and Zabbix frontend

```bash
sudo apt install zabbix-server-pgsql zabbix-frontend-php php7.3-pgsql zabbix-nginx-conf zabbix-agent
```

------

### 8. Create initial database

```bash
# Switch to the postgres user.
su - postgres

# Create a database user called zabbix
sudo -u postgres createuser --pwprompt zabbix

# Create a database with UTF8 called Zabbix and give ownership to the zabbix user created before
sudo -u postgres createdb -O zabbix -E Unicode -T template0 zabbix
```

------

### 9. import initial schema and data to the database.

```bash
sudo zcat /usr/share/doc/zabbix-server-pgsql*/create.sql.gz | sudo -u zabbix psql zabbix
```

------

### 10. Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf

```
DBPassword=password
```

------

### 11. Configure PHP for Zabbix frontend

Edit file /etc/zabbix/nginx.conf, uncomment and set 'listen' and 'server_name' directives.

```bash
# listen 80;
# server_name example.com;
```

Edit file /etc/zabbix/php-fpm.conf, uncomment and set the right timezone for you.

```php
; php_value[date.timezone] = Europe/Riga
```

------

### 12. Start Zabbix server and agent processes

```bash
systemctl restart zabbix-server zabbix-agent nginx php7.3-fpm

systemctl enable zabbix-server zabbix-agent nginx php7.3-fpm

systemctl status zabbix-server zabbix-agent nginx php7.3-fpm
```

------

### 13. Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name

