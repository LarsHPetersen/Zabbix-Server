# Zabbix Server 5.2 install with Nginx and MariaDB on CentOS 8

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
### 3. Install mariaDB

```bash
sudo dnf -y install mariadb-server

sudo systemctl enable --now mariadb

sudo systemctl status mariadb
```
------
### 4. Secure mariaDB

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
### 5. Create initial database

```bash
mysql -uroot -p
```

```sql
mariadb> create database zabbix character set utf8 collate utf8_bin;
mariadb> create user zabbix@localhost identified by 'password';
mariadb> grant all privileges on zabbix.* to zabbix@localhost;
mariadb> quit;
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
sudo dnf install zabbix-server-mysql zabbix-web-mysql zabbix-nginx-conf zabbix-agent
```
------
### 8. import initial schema and data to the database.

```bash
sudo zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```
------
### 9. Configure the database for Zabbix server

Edit file /etc/zabbix/zabbix_server.conf

Add the password you created for the [zabbix database user](###5.-Create-initial-database).

```
DBPassword=password
```
------
### 10. Configure PHP for Zabbix frontend

Open the /etc/nginx/conf.d/zabbix.conf file and uncomment the listen and server_name parameters. In the server name enter the domain name of your server or _ if you only want to access it using an IP address.

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

### 11. Set timezone

Edit file /etc/zabbix/php-fpm.conf, uncomment and set the right timezone for you.

```php
; php_value[date.timezone] = Europe/Riga
```
-------

### 12. Start Zabbix server and agent processes

```bash
# Restarts the zabbix-server, zabbix-agent, nginx and php service.
sudo systemctl restart zabbix-server zabbix-agent nginx php-fpm

# Enables the zabbix-server, zabbix-agent, nginx and php service to start automatically after a reboot.
sudo systemctl enable zabbix-server zabbix-agent nginx php-fpm

# Gets the status of zabbix-server, zabbix-agent, apache and php service.
sudo systemctl status zabbix-server zabbix-agent nginx php-fpm
```

------

### 13. open the firewall

```bash
# Opens the firewall for the service http.
sudo firewall-cmd --add-service=http --permanent

# reloads the firewall so the new rules take effect.
sudo firewall-cmd --reload
```

--------

### 14. Configure Zabbix frontend

Connect to your newly installed Zabbix frontend: http://server_ip_or_name

