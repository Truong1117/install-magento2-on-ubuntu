# install-magento2-on-ubuntu
1. Log in via SSH and update the system

sudo apt update && sudo apt upgrade

2. Install PHP

Magento 2.4.3 is fully compatible with PHP 7.4. To install the latest stable version of PHP 7.4 and all necessary modules, run:
$ sudo apt install php7.4-{bcmath,common,curl,fpm,gd,intl,mbstring,mysql,soap,xml,xsl,zip,cli}

Once completed, we can increase some PHP variable values to meet Magento’s minimum requirements.

$ sudo sed -i "s/memory_limit = .*/memory_limit = 768M/" /etc/php/7.4/fpm/php.ini
$ sudo sed -i "s/upload_max_filesize = .*/upload_max_filesize = 128M/" /etc/php/7.4/fpm/php.ini
$ sudo sed -i "s/zlib.output_compression = .*/zlib.output_compression = on/" /etc/php/7.4/fpm/php.ini
$ sudo sed -i "s/max_execution_time = .*/max_execution_time = 18000/" /etc/php/7.4/fpm/php.ini

3. Install Web server
In this tutorial, we are going to use nginx the web server, you can check our other blog posts if you want to use Apache.
  $ sudo apt install nginx

Now, we need to create an nginx server block for our domain.
$ nano /etc/nginx/sites-enabled/magento.conf

Then insert the following into the configuration file.
upstream fastcgi_backend {
server unix:/run/php/php7.4-fpm.sock;
}

server {
server_name yourdomain.com;
listen 80;
set $MAGE_ROOT /opt/magento2;
set $MAGE_MODE developer; # or production

access_log /var/log/nginx/magento2-access.log;
error_log /var/log/nginx/magento2-error.log;
}

Save the file then exit. To test the configuration file we can run this command

$ sudo nginx -t

If everything is okay and no error message, we can reload nginx.

sudo systemctl reload nginx

4. Install MariaDB Server

Magento prefered MariaDB Database Server to default MySQL Database Server, because of faster and better performance. To install MariaDB Server and Client, run this command line:
  sudo apt-get install mariadb-server mariadb-client
  
Make sure it start and startup every time you reboot server:
sudo systemctl restart mariadb.service
sudo systemctl enable mariadb.service

You’ve just installed MariaDB server, now you have to initially setup this database server.

sudo mysql_secure_installation

It prompte and you choose the following option:

Enter current password for root (enter for none): Enter
Set root password? [Y/n]: Y
New password: Type your password
Re-enter new password: Type your password
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]:  Y
Reload privilege tables now? [Y/n]:  Y

5. Create MySQL User (Required)

From Magento 2.3.x, Magento requires a unique user for Magento installation, it cannot default user: root.

First of all, you have to login to MariaDB:
  sudo mysql -u root -p

Create a new database for Magento 2:
  CREATE DATABASE magento2

Then create a new user name call: mageplaza
  CREATE USER 'mageplaza'@'localhost' IDENTIFIED BY 'YOUR_PASSWORD';
  
Grant mageplaza user to magento2 database:
  GRANT ALL ON magento2.* TO 'mageplaza'@'localhost' IDENTIFIED BY 'YOUR_PASSWORD' WITH GRANT OPTION;
  
Ok, time to flush privileges and exit.
  FLUSH PRIVILEGES;
  EXIT;
  
6. Install Composer

Download Composer and install or you can use command line to install Composer
  curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
  
Check Composer installed or not just type:
  composer -v

7. Install Magento 2
