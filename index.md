# Magento 2 (version 2.4.4) Installation Guide:

## Introduction
Magento is one of the most popular open-source e-commerce platform written in PHP powered by Zend framework. In this tutorial, you will learn the steps required to install and setup Magento v2.4.4 on your Linux/Ubuntu (ubuntu 20.04 & ubuntu 22.04) server.

## Prerequisites
- Ubuntu 20.04 or 22.04
- PHP 8.1+
- MySQL 8.0+
- Apache2
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

### Step 10: Installing Magento using composer:
Change to our web directory
```
cd /var/www/html
```
Run the composer command to install Magento
```
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition magento
```
You will be prompted to provide a username password for this. Put your public key as the username and the private key as the password. It will take some time for the Composer to download and install all the required dependencies so **be patient.**  
  
Once the download is done, its time to change to magento directory and continue with the Installation:  
```
cd magento/
```
Run the Following Command (Make sure to change the username, password and database name from _**Step 6**_
```
php bin/magento setup:install --base-url=http://localhost --db-host=localhost --db-name=magento --db-user=magento --db-password=magento123 --admin-firstname=First --admin-lastname=Last --admin-email=test@example.com --admin-user=admin --admin-password=admin123 --language=en_US --currency=USD --timezone=America/Chicago --use-rewrites=1 --backend-frontname=admin
```
### Step 11: Setup Cron jobs:
Cronjob is nothing but a scheduler that runs a specified file path at a specified interval. Magento needs some of these Cron Jobs to be set up in order to function properly.  
  
To add these Crons follow this command  
Change to magento Directory
```
cd /var/www/html/magento
```
Install the Cron:
```
sudo php bin/magento cron:install
```
You can check the installed Magento Cron Jobs by using this command
```
sudo crontab -l
```
You should find theses settings in your Cron
```
* * * * * /usr/bin/php /var/www/html/magento/bin/magento cron:run 2>&1 | grep -v "Ran jobs by schedule" >> /var/www/html/magento/var/log/magento.cron.log  * * * * * /usr/bin/php /var/www/html/magento/update/cron.php >> /var/www/html/magento/var/log/update.cron.log  * * * * * /usr/bin/php /var/www/html/magento/bin/magento setup:cron:run >> /var/www/html/magento/var/log/setup.cron.log
```
  
## Common Errors & Exceptions
If you are getting 500 error after the installation please follow these steps:

This problem is probably due to directory access problems. To solve that use the following commands:
```
sudo chown -R www-data:www-data /var/www/html/magento
sudo chmod -R 755/var/www/html/magento
```
If you have problems logging into the admin panel, disable the 2FA Module (not needed for localhost installations)  
```
sudo bin/magento module:disable Magento_TwoFactorAuth
```
If you are still getting errors you can debug those errors by referring the following logs
```
tail -f /var/log/apache2/magento_error.log $ tail -f /var/log/apache2/magento_access.log
```
And with this, your Magento setup is up and ready to be used for your E-Commerce site.

## Conclusion
By following the above-mentioned steps you are now ready to create your e-commerce platform with a working Magento installation. In case you have encountered something, which is not covered as a part of this tutorial, then please feel free to contact me at ather90@gmail.com
