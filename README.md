# Zabbix-Native
Install Zabbix Server With Seperate Database

# Install Zabbix Server with External Database

## Prerequisites
- Two Ubuntu 22.04 servers:
  - **Server 1:** Zabbix Server + Frontend
  - **Server 2:** MariaDB (Database)
- Root or sudo access on both servers
- Network connectivity between the servers (port 3306 open for MariaDB)

---

## Step 1: Prepare the Servers
On both servers:

```bash
sudo apt update
sudo apt upgrade -y
sudo timedatectl set-timezone Asia/Tehran
```

---

## Step 2: Install and Configure MariaDB on the Database Server

### 2.1 Install MariaDB
```bash
sudo apt install mariadb-server -y
```

### 2.2 Secure MariaDB
```bash
sudo mysql_secure_installation
```
Recommended answers:
- Set root password: **Yes**
- Remove anonymous users: **Yes**
- Disallow root login remotely: **No**
- Remove test database: **Yes**
- Reload privilege tables: **Yes**

### 2.3 Create Database and User for Zabbix
```bash
sudo mysql -u root -p
```
Inside MariaDB prompt:

```sql
CREATE DATABASE zabbix character set utf8mb4 collate utf8mb4_bin;
CREATE USER 'zabbix'@'%' IDENTIFIED BY 'YourStrongPassword';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'%';
FLUSH PRIVILEGES;
EXIT;
```

> Replace `YourStrongPassword` with a strong password.

### 2.4 Allow Remote Connections
Edit MariaDB config:

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
Find and change:

```diff
-bind-address = 127.0.0.1
+bind-address = 0.0.0.0
```

Restart MariaDB:

```bash
sudo systemctl restart mariadb
```

### 2.5 Open Firewall for Database Server (Optional)

```bash
sudo ufw allow from <Zabbix-Server-IP> to any port 3306
```

---

## Step 3: Install Zabbix Server on Application Server

### 3.1 Add Zabbix Repository
```bash
wget https://repo.zabbix.com/zabbix/6.5/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.5-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.5-1+ubuntu22.04_all.deb
sudo apt update
```

### 3.2 Install Zabbix Server, Frontend, and Agent
```bash
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent -y
```

### 3.3 Import Initial Schema and Data
```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql -u zabbix -p -h <Database-Server-IP> zabbix
```

---

## Step 4: Configure Zabbix Server

Edit Zabbix server config:

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Update the following values:

```ini
DBHost=<Database-Server-IP>
DBName=zabbix
DBUser=zabbix
DBPassword=YourStrongPassword
```

---

## Step 5: Configure PHP

Edit Apache Zabbix config:

```bash
sudo nano /etc/zabbix/apache.conf
```

Set your timezone:

```ini
php_value date.timezone Asia/Tehran
```

---

## Step 6: Start and Enable Services
```bash
sudo systemctl restart zabbix-server zabbix-agent apache2
sudo systemctl enable zabbix-server zabbix-agent apache2
```

---

## Step 7: Complete Installation via Web Interface
- Open your browser and navigate to:

```text
http://<Zabbix-Server-IP>/zabbix
```

- Follow the installation wizard.
- Use the database information you created earlier.

---

## Default Zabbix Login Credentials

| Field | Value |
|:-----|:------|
| Username | Admin |
| Password | zabbix |

> Note: `Admin` with capital `A`.

---

## How to Unlock Account if Blocked (Optional)

If you see `account is temporarily blocked` error, connect to the database server:

```bash
sudo mysql -u root -p
```

Then:

```sql
USE zabbix;
UPDATE users SET attempt_failed = 0, attempt_clock = 0 WHERE alias='Admin';
```

---

## Done! 🌟

Now your Zabbix server is ready!

---

