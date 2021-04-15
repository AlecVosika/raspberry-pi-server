# **Description**

- [owncloud installation](https://doc.owncloud.com/server/admin_manual/installation/quick_guides/ubuntu_20_04.html)

## **expected procedure**

1. follow [owncloud installation](https://doc.owncloud.com/server/admin_manual/installation/quick_guides/ubuntu_20_04.html) guide.
2. follow begining parts of [nginx guide](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)
3. use [this link](https://www.digitalocean.com/community/tutorials/how-to-configure-nginx-as-a-web-server-and-reverse-proxy-for-apache-on-one-ubuntu-20-04-server) and [this link](https://phoenixnap.com/kb/how-to-install-nginx-on-ubuntu-20-04) to fill in the blanks

## **Preparation**

```Shell Scripting
apt update && \
  apt upgrade -y
```

```Shell Scripting
FILE="/usr/local/bin/occ"
/bin/cat <<EOM >$FILE
#! /bin/bash
cd /var/www/owncloud
sudo -u www-data /usr/bin/php /var/www/owncloud/occ "\$@"
EOM
```

```Shell Scripting
chmod +x /usr/local/bin/occ
```

```Shell Scripting
apt install -y \
  apache2 \
  libapache2-mod-php \
  mariadb-server \
  openssl \
  php-imagick php-common php-curl \
  php-gd php-imap php-intl \
  php-json php-mbstring php-mysql \
  php-ssh2 php-xml php-zip \
  php-apcu php-redis redis-server \
  wget
```

```Shell Scripting
add-apt-repository ppa:ondrej/php
apt-get update
```

```Shell Scripting
apt-key adv --recv-keys --keyserver hkps://keyserver.ubuntu.com:443 4F4EA0AAE5267A6C
```

```Shell Scripting
apt install -y \
  ssh bzip2 rsync curl jq \
  inetutils-ping smbclient\
  php-smbclient coreutils php-ldap
```

```Shell Scripting
sed -i "s#html#owncloud#" /etc/apache2/sites-available/000-default.conf
service apache2 restart
```

```Shell Scripting
FILE="/etc/apache2/sites-available/owncloud.conf"
/bin/cat <<EOM >$FILE
Alias /owncloud "/var/www/owncloud/"

<Directory /var/www/owncloud/>
  Options +FollowSymlinks
  AllowOverride All

 <IfModule mod_dav.c>
  Dav off
 </IfModule>

 SetEnv HOME /var/www/owncloud
 SetEnv HTTP_HOME /var/www/owncloud
</Directory>
EOM
```

```Shell Scripting
a2ensite owncloud.conf
service apache2 reload
```

```Shell Scripting
mysql -u root -e "CREATE DATABASE IF NOT EXISTS owncloud; \
GRANT ALL PRIVILEGES ON owncloud.* \
  TO owncloud@localhost \
  IDENTIFIED BY 'password'";
```

```Shell Scripting
echo "Enabling Apache Modules"
a2enmod dir env headers mime rewrite setenvif
service apache2 reload
```

```Shell Scripting
cd /var/www/
wget https://download.owncloud.org/community/owncloud-10.7.0.tar.bz2 && \
tar -xjf owncloud-10.7.0.tar.bz2 && \
chown -R www-data. owncloud
```

```Shell Scripting
occ maintenance:install \
    --database "mysql" \
    --database-name "owncloud" \
    --database-user "owncloud" \
    --database-pass "password" \
    --admin-user "admin" \
    --admin-pass "admin"
```

```Shell Scripting
myip=$(hostname -I|cut -f1 -d ' ')
occ config:system:set trusted_domains 1 --value="$myip"
```

```Shell Scripting
occ background:cron
```

```Shell Scripting
echo "*/15  *  *  *  * /var/www/owncloud/occ system:cron" \
  > /var/spool/cron/crontabs/www-data
chown www-data.crontab /var/spool/cron/crontabs/www-data
chmod 0600 /var/spool/cron/crontabs/www-data
```

```Shell Scripting
echo "*/15 * * * * /var/www/owncloud/occ user:sync 'OCA\User_LDAP\User_Proxy' -m disable -vvv >> /var/log/ldap-sync/user-sync.log 2>&1" >> /var/spool/cron/crontabs/www-data
chown www-data.crontab  /var/spool/cron/crontabs/www-data
chmod 0600  /var/spool/cron/crontabs/www-data
mkdir -p /var/log/ldap-sync
touch /var/log/ldap-sync/user-sync.log
chown www-data. /var/log/ldap-sync/user-sync.log
```

```Shell Scripting
occ config:system:set \
   memcache.local \
   --value '\OC\Memcache\APCu'
occ config:system:set \
   memcache.locking \
   --value '\OC\Memcache\Redis'
occ config:system:set \
   redis \
   --value '{"host": "127.0.0.1", "port": "6379"}' \
   --type json
```

```Shell Scripting
FILE="/etc/logrotate.d/owncloud"
sudo /bin/cat <<EOM >$FILE
/var/www/owncloud/data/owncloud.log {
  size 10M
  rotate 12
  copytruncate
  missingok
  compress
  compresscmd /bin/gzip
}
EOM
```

```Shell Scripting
cd /var/www/
chown -R www-data. owncloud
```

```Shell Scripting

```

```Shell Scripting

```

```Shell Scripting

```

```Shell Scripting

```

```Shell Scripting

```
