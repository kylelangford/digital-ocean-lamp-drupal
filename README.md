# Drupal 9 on Ubuntu 20.04 using Digital Ocean

### Hosting - Digital Ocean

- 1 Step Droplet - **Use Ubuntu with LAMP**
- Use **Password Auth** to avoid issues with SSH
- SSH in as `ssh root@64.225.52.153`

**Full Guide:** (1 Step LAMP skips to step 4)
https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04 

### Create subdomains

Digital Ocean - Add A Records
DNS Registrar - Create A Records

    www.sub.mydomain.com
    sub.mydomain.com


### Create Linux user 

This will create two users with sudo privileges, switch to user.

```bash
adduser orion
usermod -aG sudo orion
su - orion
```

reference: https://www.digitalocean.com/community/tutorials/how-to-create-a-new-sudo-enabled-user-on-ubuntu-18-04-quickstart

### Create Database and assign new MySql user

Create two databases and a new MySql user. Assign the user full privalages to each database. 

The MySQL root password is in `~/.digitalocean_password.`

```sql
sudo mysql -u root -p
CREATE DATABASE client_orion;
CREATE DATABASE content_orion;
CREATE USER 'linkwell'@'localhost' IDENTIFIED BY 'linkwell';
GRANT ALL PRIVILEGES ON client_orion . * TO 'linkwell'@'localhost';
GRANT ALL PRIVILEGES ON content_orion . * TO 'linkwell'@'localhost';
EXIT;
```

reference: https://www.digitalocean.com/community/tutorials/how-to-create-and-manage-databases-in-mysql-and-mariadb-on-a-cloud-server

### Configure VHOST 

Create two directories in `/var/www/`

```bash
sudo mkdir content-orion
sudo mkdir client-orion
sudo chown -R $USER:$USER  client-orion
sudo chown -R $USER:$USER  content-orion
```

Create configuration files `/etc/apache2/sites-availlable/`

```
sudo nano content-orion.linkwellhealth.com.conf
```
    <VirtualHost *:80>
	    ServerName content-orion.linkwellhealth.com
	    ServerAlias www.content-orion.linkwellhealth.com
	    DocumentRoot /var/www/content-orion.linkwellhealth.com/drupal/web
    </VirtualHost>

```
sudo nano client-orion.linkwellhealth.com.conf
``` 

    <VirtualHost *:80>
        ServerName client-orion.linkwellhealth.com
        ServerAlias client-orion.linkwellhealth.com
        DocumentRoot /var/www/client-orion.linkwellhealth.com/drupal/web
    </VirtualHost>

Enable sites, will create a symlink in  `/etc/apache2/sites-enabled/`
```
sudo a2ensite content-orion.linkwellhealth.com.conf
sudo a2ensite client-orion.linkwellhealth.com.conf
sudo systemctl restart apache2
```

reference: https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-ubuntu-20-04 (step 4)

### SSL 

This is the easiest and recomended way to install SSL and automatically edit the vhost configuration files.

`sudo certbot --apache`

**note** - Choose redirect

> To use Certbot, you’ll need a registered domain name and two DNS records:
    * An A record from a domain (e.g., example.com) to the server’s IP address
    * An A record from a domain prefaced with www (e.g., www.example.com) to the server’s IP address

reference: https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-18-04 (Start at 4)

### Install Drupal Dependencies

Feedback from initial test install showed dom, xml, SimpleXML, and mbstring php modules are missing. You can resolve this by running the following. (dom with install xml, SimpleXML)

```
sudo apt-get install php8.0-dom
sudo apt-get install php8.0-mbstring
sudo systemctl restart apache2
```

**Note** - For some reason the php cli version and the apache version are different I found the easiest way to fix this was to set the cli version to 8.0 and install modules for 8.0 instead of 8.1. (should match what phpinfo() shows.)

`sudo update-alternatives --config php`

### Install Composer 

reference: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-composer-on-ubuntu-20-04

### Install Drupal 9 with composer

Download and Install Dependencies

`composer create-project drupal/recommended-project drupal`

**note** - Fix write permissions by assigning the correct group 

`sudo chown -R orion:www-data sites/default/` 

```console
# Manual Steps
sudo cp default.settings.php settings.php
sudo chmod 777 settings.php
```

reference: https://www.drupal.org/docs/develop/using-composer/using-composer-to-install-drupal-and-manage-dependencies

### Fix Clean URLs

In order to fix clean URLs add this to the ssl conf(*443) for both sites 

`client-orion.linkwellhealth.com.conf`

    <Directory /var/www/client-orion.linkwellhealth.com/drupal/web/>
        AllowOverride All
    </Directory>

`content-orion.linkwellhealth.com.conf`

    <Directory /var/www/content-orion.linkwellhealth.com/drupal/web/>
       AllowOverride All
    </Directory>

### Run Install.php

1. Complete Prompts
2. 
3. Profit!
