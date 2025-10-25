# Omeka Classic Installation - Technical Documentation

**Server:** DigitalOcean Ubuntu 22.04 | **IP:** your_ipaddress | **Date:** October 25, 2025

---

## 1. Installation Overview

**Software Stack:**
- Apache 2.4.52
- PHP 8.1.x with required extensions
- MariaDB 10.6.x
- ImageMagick 6.9.x
- Omeka Classic 3.1.2

---

## 2. Installation Steps

### 2.1 Initial Setup
```bash
# Connect via SSH
ssh root@yourip_address

# Update system
apt update && apt upgrade -y
```

### 2.2 Install Apache
```bash
apt install apache2 -y
systemctl start apache2
systemctl enable apache2
```

### 2.3 Install MySQL/MariaDB
```bash
apt install mariadb-server -y
systemctl start mariadb
systemctl enable mariadb
mysql_secure_installation  # Set root password, remove test data
```

### 2.4 Install PHP & Extensions
```bash
apt install php libapache2-mod-php php-mysql php-xml php-mbstring \
            php-curl php-zip php-gd php-intl php-json -y
```

### 2.5 Install ImageMagick
```bash
apt install imagemagick php-imagick -y
systemctl restart apache2
```

### 2.6 Create Database & User
```bash
mysql -u root -p
```
```sql
CREATE DATABASE omeka CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'omekauser'@'localhost' IDENTIFIED BY 'OmekaPass2024!';
GRANT ALL PRIVILEGES ON omeka.* TO 'omekauser'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

### 2.7 Download & Deploy Omeka
```bash
cd /tmp
apt install unzip -y
wget https://github.com/omeka/Omeka/releases/download/v3.1.2/omeka-3.1.2.zip
unzip omeka-3.1.2.zip
mv Omeka /var/www/html/omeka
```

### 2.8 Set Permissions
```bash
chown -R www-data:www-data /var/www/html/omeka
chmod -R 755 /var/www/html/omeka
mkdir -p /var/www/html/omeka/files
chmod -R 777 /var/www/html/omeka/files
```

### 2.9 Configure Database Connection
```bash
cd /var/www/html/omeka
cp db.ini.changeme db.ini
nano db.ini
```
**Edit to:**
```ini
[database]
host     = "localhost"
username = "omekauser"
password = "OmekaPass2024!"
dbname   = "omeka"
prefix   = "omeka_"
charset  = "utf8mb4"
```

### 2.10 Configure Apache Virtual Host
```bash
nano /etc/apache2/sites-available/omeka.conf
```
```apache
<VirtualHost *:80>
    ServerName your_ipaddress
    DocumentRoot /var/www/html/omeka
    <Directory /var/www/html/omeka>
        AllowOverride All
        Require all granted
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/omeka_error.log
</VirtualHost>
```

### 2.11 Enable Site & Modules
```bash
a2enmod rewrite
a2ensite omeka.conf
systemctl reload apache2
```

### 2.12 Complete Web Installation
Access: `http://your_ipaddress/omeka/install/install.php`
- Fill in admin credentials
- Set ImageMagick path: `/usr/bin`
- Complete installation form

---

## 3. Troubleshooting Issues & Solutions

### Issue 1: Database Permission Denied
**Error:** `ERROR 1044: Access denied for user 'omekauser'@'localhost'`

**Cause:** Insufficient database privileges

**Solution:**
```sql
mysql -u root -p
DROP USER IF EXISTS 'omekauser'@'localhost';
CREATE USER 'omekauser'@'localhost' IDENTIFIED BY 'OmekaPass2024!';
GRANT ALL PRIVILEGES ON omeka.* TO 'omekauser'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

**Verification:**
```bash
mysql -u omekauser -pOmekaPass2024! omeka
# Should connect successfully
```

---

### Issue 2: Unknown Database
**Error:** `ERROR 1049: Unknown database 'omeka'`

**Cause:** Database not created

**Solution:**
```sql
CREATE DATABASE omeka CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

---

### Issue 3: ImageMagick Not Found
**Error:** Installation form requires ImageMagick path

**Cause:** ImageMagick not installed

**Solution:**
```bash
apt install imagemagick php-imagick -y
which convert  # Verify: /usr/bin/convert
systemctl restart apache2
```
**Configuration:** Enter `/usr/bin` in installation form

---

### Issue 4: Generic Error Message
**Error:** "Omeka has encountered an error" with no details

**Cause:** Production mode hides error details

**Solution:**
```bash
nano /var/www/html/omeka/.htaccess
# Change: SetEnv APPLICATION_ENV production
# To: SetEnv APPLICATION_ENV development
systemctl restart apache2
```
**Note:** Change back to `production` after installation

---

### Issue 5: File Upload Errors
**Error:** Permission denied on file uploads

**Solution:**
```bash
chmod -R 777 /var/www/html/omeka/files
chown -R www-data:www-data /var/www/html/omeka/files
```

---

## 4. Post-Installation Security

```bash
# Disable debug mode
nano /var/www/html/omeka/.htaccess
# Set: SetEnv APPLICATION_ENV production

# Secure database config
chmod 600 /var/www/html/omeka/db.ini

# Configure firewall
ufw allow 22/tcp
ufw allow 80/tcp
ufw enable
```

---

## 5. Verification Commands

```bash
# Check services
systemctl status apache2
systemctl status mariadb

# Test database
mysql -u omekauser -p omeka -e "SHOW TABLES;"

# View logs
tail -f /var/log/apache2/omeka_error.log
tail -f /var/www/html/omeka/application/logs/errors.log
```

---

## 6. Access Points

| Resource | URL |
|----------|-----|
| **Public Site** | http://your_ipaddress/omeka |
| **Admin Panel** | http://your_ipaddress/omeka/admin |
