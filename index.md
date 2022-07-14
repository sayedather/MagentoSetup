# Magento 2 Installation Steps:

## Introduction
Magento is one of the most popular open-source e-commerce platform written in PHP powered by Zend framework. In this tutorial, you will learn the steps required to install and setup Magento on your Linux/Ubuntu server.

## Prerequisites
- PHP 8.1+
- MySQL 8.0+
- Apache
- Composer 2
- Elastic Search 7.x

## Installation Steps
### Step 1: First step is to install Apache2
To install and setup Apache use the following commands:
```
sudo apt install apache2
sudo systemctl start apache2
sudo systemctl enable apache2
```
### Step 2: Create a new Apache virtual host and configure it.
Create a new configuration file name magento.conf
```
sudo nano /etc/apache2/sites-available/magento.conf

```
Paste the following code into the newly created config file
```
<VirtualHost *:80>
     ServerAdmin admin@domain.com
     DocumentRoot /var/www/html/magento/
     ServerName domain.com
     ServerAlias www.domain.com

     <Directory /var/www/html/magento/>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/magento_error.log
     CustomLog ${APACHE_LOG_DIR}/magento_access.log combined
</VirtualHost>

```

### Step 3: Enable Rewrite mod and enable the new site
```
 sudo a2ensite magento.conf
 sudo a2dissite 000-default.conf
 sudo a2enmod rewrite
 sudo systemctl reload apache2
 
 test the newly made website in your browser by visiting http://localhost and you should get an Apache Installation Page, This means its working
```
### Step 4: Installing PHP
Use the below command to install PHP:
```
sudo apt install lsb-release ca-certificates apt-transport-https software-properties-common -y
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update
sudo apt install php8.1
sudo apt-get install php-common php-cli php-curl php-intl php-zip -y

```
### Step 5: Installing PHP Dependencies:
```
sudo apt-get install php8.1-curl php8.1-opcache php8.1-pdo php8.1-calendar php8.1-ctype php8.1-dom php8.1-exif php8.1-ffi php8.1-fileinfo php8.1-ftp php8.1-gettext php8.1-iconv php8.1-mbstring php8.1-phar php8.1-posix php8.1-readline php8.1-shmop php8.1-simplexml php8.1-sockets php8.1-sysvmsg php8.1-sysvsem php8.1-sysvshm php8.1-tokenizer php8.1-xmlreader php8.1-xmlwriter php8.1-xsl php8.1-soap php8.1-bcmath php8.1-intl php8.1-gd php8.1-mysql php8.1-zip
```
Restart Apache to reflect the new changes:
```
sudo systemctl restart apache2
```

### Step 6:  Setting up MySQL Server
Install MySQL server (make sure to change the username, password and the database name)
```
sudo apt install mysql-server
sudo systemctl start mysql.service
sudo mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'yourpassword';
exit
sudo mysql_secure_installation
CREATE USER 'magento'@'localhost' IDENTIFIED BY 'magento123';
CREATE DATABASE magento COLLATE utf8_general_ci;
GRANT ALL PRIVILEGES ON *.* TO 'magento'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### Step 7: Install composer
You need Composer to install all the Magento and its dependencies
Use the below command to install composer globally on your Ubuntu system:
```
cd /tmp
sudo curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```
### Step 8: Acquire / Generate secret keys from your Magento market place user account.
In order to install Magento in the next step, you will have to specify Public and Private keys from your Magento market-place account. 
If you haven't registered yet you can register [here](https://account.magento.com/applications/customer/create/).

 - Login to Magento Marketplace [here](https://marketplace.magento.com/).
 - Click on your username dropdown in the top navbar > My Profile
 - Click on [Access Keys]
 - Click on [Create A New Access Key]

### Step 9: Install Elastic Search:
```
sudo apt install curl
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install elasticsearch
```
Now we need to change the Elastic Search Configuration file to listen to localhost connections
```
sudo nano /etc/elasticsearch/elasticsearch.yml
```
Uncomment this line ``#network.host: 192.168.0.1`` and change it to ``network.host: localhost``  
  
Next we start Elastic Search by
```
sudo systemctl start elasticsearch
```
and then we enable start on boot
```
sudo systemctl enable elasticsearch 
```
 To test Elastic Search Installation, run this code:  
 ```
 curl -X GET 'http://localhost:9200'
 ```
