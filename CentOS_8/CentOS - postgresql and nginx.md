# Zabbix Server 5.2 install with Nginx and PostgreSQL on CentOS 8

### 1. Update the system

```bash
sudo dnf update
```

------

### 2. Install nginx

```bash
sudo dnf install nginx
```

------

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

------

### 4. Add a password to the Postgres user.

```bash
sudo passwd postgres
```

------

### 5. Add the postgres user to the sudo group

```bash
sudo usermod -aG wheel postgres
```

------

### 6. Install Zabbix repository

```bash
sudo rpm -Uvh https://repo.zabbix.com/zabbix/5.2/rhel/8/x86_64/zabbix-release-5.2-1.el8.noarch.rpm

sudo dnf clean all
```

------

### 7. Install Zabbix Server, Zabbix Agent and Zabbix frontend

```bash
sudo dnf install zabbix-server-pgsql zabbix-web-pgsql zabbix-nginx-conf zabbix-agent
```

------

### Create initial database

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

### 10. Change the pg_hba.conf config fil from ident to password.

Change the configuration in `/var/lib/pgsql/12/data/pg_hba.conf` from ident to password.

Before
```bash
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             0.0.0.0/0               ident
# IPv6 local connections:
host    all             all             ::1/128                 ident
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
```

After
```bash
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     password
# IPv4 local connections:
host    all             all             0.0.0.0/0               password
# IPv6 local connections:
host    all             all             ::1/128                 password
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
```

------

### 11. Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf

Add the password you created for the [zabbix database user](###-create-initial-database).

```
DBPassword=password
```

------

### 12. Configure PHP for Zabbix frontend

Edit file /etc/nginx/conf.d/zabbix.conf, uncomment and set 'listen' and 'server_name' directives.

```bash
# listen 80;
# server_name example.com;
```

------

if you want to access the zabbix server with the IP address, you also need to comment out the entire server section in the /etc/nginx/nginx.conf file.

```conf
#    server {
#        listen       80 default_server;
#        listen       [::]:80 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }
```

> If your using vim, to comment out multiple lines.

1. First, go to the first line you want to comment, press **`Ctrl + V`**. This will put the editor in the VISUAL BLOCK mode.

2. Then using the arrow key and select until the last line

3. Now press **`Shift + I`**, which will put the editor in INSERT mode and then press #. This will add a hash to the first line.

4. Then press **`Esc`** (give it a second), and it will insert a # character on all other selected lines.

5. save and exit the file by typing **`:wq`**

------

Edit file /etc/php-fpm.d/zabbix.conf, uncomment and set the right timezone for you.

```php
; php_value[date.timezone] = Europe/Riga
```

### 13. Start Zabbix server and agent processes

```bash
# Restarts the zabbix-server, zabbix-agent, nginx and php service.
sudo systemctl restart zabbix-server zabbix-agent nginx php-fpm

# Enables the zabbix-server, zabbix-agent, nginx and php service to start automatically after a reboot.
sudo systemctl enable zabbix-server zabbix-agent nginx php-fpm

# Gets the status of zabbix-server, zabbix-agent, nginx and php service.
sudo systemctl status zabbix-server zabbix-agent nginx php-fpm
```

------

### 14. open the firewall

```bash
# Opens the firewall for the service http.
sudo firewall-cmd --add-service=http --permanent

# reloads the firewall so the new rules take effect.
sudo firewall-cmd --reload
```

--------
### 15. Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name
