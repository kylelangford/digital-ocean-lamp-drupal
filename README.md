Linkwell

Steps

1. Create Subdomains
2. Use ubuntu with LAMP (1 Step sets up PHP, MYSQL, Firewall)
3. Use Password Auth (set root pw)
4. SSH in as Root
5. Create new user (https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04) (https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-ubuntu-18-04-quickstart)

adduser orion
adduser lynx
       usermod -aG sudo orion
usermod -aG sudo lynx su - orion

6. Create Database (x2)(Client, Content) (https://www.digitalocean.com/community/tutorials/how-to-create-and-manage-databases-in-mysql-and-mariadb-on-a-cloud-server)
The MySQL root password is in ~/.digitalocean_password.

sudo mysql -u root -p
CREATE DATABASE client-orion;
CREATE DATABASE content-orion;
CREATE USER 'linkwell'@'localhost' IDENTIFIED BY 'linkwell';
GRANT ALL PRIVILEGES ON client_orion . * TO 'linkwell'@'localhost';
GRANT ALL PRIVILEGES ON content_orion . * TO 'linkwell'@'localhost';
EXIT;

7.  Configure VHOST (MANUAL) (step 4 https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04)

	sudo mkdir /var/www/content-orion
	sudo mkdir /var/www/client-orion 	sudo chown -R $USER:$USER  /var/www/client-orion
 sudo chown -R $USER:$USER  /var/www/content-orion

 sudo nano /etc/apache2/sites-availlable/content-orion.linkwellhealth.com.conf

<VirtualHost *:80>
        ServerName content-orion.linkwellhealth.com
        ServerAlias www.content-orion.linkwellhealth.com
        DocumentRoot /var/www/content-orion.linkwellhealth.com/drupal/web
</VirtualHost>

 sudo nano /etc/apache2/sites-availlable/client-orion.linkwellhealth.com.conf

<VirtualHost *:80>
        ServerName client-orion.linkwellhealth.com
        ServerAlias client-orion.linkwellhealth.com
        DocumentRoot /var/www/client-orion.linkwellhealth.com/drupal/web
</VirtualHost>

sudo a2ensite content-orion.linkwellhealth.com.conf
sudo a2ensite client-orion.linkwellhealth.com.conf
sudo systemctl restart apache2

8. SSL Cert (Start at 4)(https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-18-04)  sudo certbot --apache (choose redirect) 
To use Certbot, you’ll need a registered domain name and two DNS records:
    * An A record from a domain (e.g., example.com) to the server’s IP address
    * An A record from a domain prefaced with www (e.g., www.example.com) to the server’s IP address

        Afterwards in order to fix clean URLs add this to the ssl conf(*443) for both sites 

  <Directory /var/www/client-orion.linkwellhealth.com/drupal/web/>
     AllowOverride All
  </Directory>

  <Directory /var/www/content-orion.linkwellhealth.com/drupal/web/>
     AllowOverride All
  </Directory>

8. Install Drupal Dependencies

# for some reason the php cli version and the apache version are different I found the easiest way to fix this was to set the cli version to 8.0 and install modules for 8.0 instead of 8.1

sudo update-alternatives --config php
(Select 8.0 or option 3)

sudo apt-get install php8.0-dom
# sudo apt-get install php8.0-gd
sudo apt-get install php8.0-mbstring
sudo systemctl restart apache2

9. Install Composer (https://www.digitalocean.com/community/tutorials/how-to-install-and-use-composer-on-ubuntu-20-04)
10. Install Drupal 9 with composer (https://www.drupal.org/docs/develop/using-composer/using-composer-to-install-drupal-and-manage-dependencies)

sudo chown -R orion:www-data default/
sudo cp default.settings.php settings.php
sudo chmod 777 settings.php (after -> sudo chmod 755 settings.php)
11. Install.php
db: client-orion
db: content-orion

=========================================================

Droplet - Drupal Test Env

ip: 64.225.52.153
http://64.225.52.153/info.php

linux: root  
p: s3cur3T3st

linux: orion
p: Qtj3p8ZhpanQ7Baa

linux: lynx
p: Qtj3p8ZhpanQ7Baa

mysql: root
p: 80a6c5d8215dc8201bbe763c5c5b266327eac021f9acf825

mysql: linkwell 
p: linkwell

http://content-orion.linkwellhealth.com/
https://client-orion.linkwellhealth.com/

U: linkwell
p: Candidate!


https://github.com/Linkwellhealthinc/content-orion
https://github.com/Linkwellhealthinc/client-orion

Have User
  git config --global user.email "you@example.com"
  git config --global user.name "Your Name"
