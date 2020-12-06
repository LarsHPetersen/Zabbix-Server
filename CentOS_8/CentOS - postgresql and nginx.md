# Zabbix Server 5.2 install with Nginx and PostgreSQL on CentOS 8

### 1. Update the system

```bash
sudo dnf update
```

### 2. Install nginx

```bash
sudo dnf install nginx
```

### 3. Install postgresql

```bash
# Install the Postgresql 12 repository
sudo rpm -Uvh https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

dnf clean all

# Disable the pre-installed DBMS module.
sudo dnf -qy module disable postgresql

# Install the postgresql 12 and postgresql12 server
sudo dnf install postgresql12 postgresql12-server
```

Initialize the database

```
sudo /usr/pgsql-12/bin/postgresql-12-setup initdb
```

start and enable the postgresql-12 service

```
sudo systemctl enable --now postgresql-12
```

### 4. Add a password to the Postgres user.

```bash
sudo passwd postgres
```

### 5. Add the postgres user to the sudo group

```bash
sudo usermod -aG wheel postgres
```

### 6. Install Zabbix repository

```bash
sudo rpm -Uvh https://repo.zabbix.com/zabbix/5.2/rhel/8/x86_64/zabbix-release-5.2-1.el8.noarch.rpm

sudo dnf clean all
```

### 7. Install Zabbix Server, Zabbix Agent and Zabbix frontend

```bash
sudo dnf install zabbix-server-pgsql zabbix-web-pgsql zabbix-nginx-conf zabbix-agent
```

### 8. Create initial database

```bash
# Switch to the postgres user.
su - postgres

# Create a database user called zabbix
sudo -u postgres createuser --pwprompt zabbix

# Create a database with UTF8 called Zabbix and give ownership to the zabbix user created before
sudo -u postgres createdb -O zabbix -E Unicode -T template0 zabbix
```

### 9. import initial schema and data to the database.

```bash
sudo zcat /usr/share/doc/zabbix-server-pgsql*/create.sql.gz | sudo -u zabbix psql zabbix
```

### 10. Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf

```
DBPassword=password
```

### 11. Configure PHP for Zabbix frontend

Edit file /etc/nginx/conf.d/zabbix.conf, uncomment and set 'listen' and 'server_name' directives.

```bash
# listen 80;
# server_name example.com;
```

Edit file /etc/zabbix/php-fpm.conf, uncomment and set the right timezone for you.

```php
; php_value[date.timezone] = Europe/Riga
```

### 12. Start Zabbix server and agent processes

```bash
sudo systemctl restart zabbix-server zabbix-agent nginx php-fpm

sudo systemctl enable zabbix-server zabbix-agent nginx php-fpm

sudo systemctl status zabbix-server zabbix-agent nginx php-fpm
```

### 13. Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name


#FIXME frontend can't connect to database.

