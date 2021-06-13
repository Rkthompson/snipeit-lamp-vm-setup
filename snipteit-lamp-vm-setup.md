# Snipe-IT VirtualBox Deployment

# Summary
This doc walks through the steps needed to setup a working install of Snipe-IT asset management using VirtualBox, Ubuntu, Apache, MySQL, and PHP.  

The first half steps through the mouse driven menus for VirtualBox and Ubuntu.  The second-half completes the configuration using the Linux CLI.


# Resources
Links to the software used.
* [Snipe-IT](https://snipeitapp.com)
* [VirtualBox](https://www.virtualbox.org)
* [Ubuntu](https://ubuntu.com)
* [Apache](https://www.apache.org)
* [MySQL](https://www.mysql.com)
* [PHP](https://www.php.net)
* [Composer](https://getcomposer.org)

# Steps

## Download OS ISO
The first step is to download a Ubuntu ISO to use as your server OS.

The latest release of Ubuntu with long term support (LTS) is available here: 
[Ubuntu Download](https://ubuntu.com/download/desktop)

This example uses Ubuntu 20.04.02 LTS.  

*Note, the desktop version is being used in this example to assist team members that aren't as strongly versed in the use of the CLI or Linux. Substitute it with the server version as desired.* 

---

## Create a Virtual Machine

Download and install VirtualBox if it is not already installed.  

The latest release of VirtualBox can be found at: 
[VirtualBox Download](https://www.virtualbox.org/wiki/Downloads)

This example uses VirtualBox 6.1.

Once installed and running click on **New** to create a new VM.
* Populate **Name**: *SnipeITServer*
* Change **Type** dropdown to: **Linux**
* Click **Next**

Set the VM memory allocation.
* Set to 2048 MB
* Click **Next**

Setup hard disk. 
* Select **Create a virtual hard disk now**
* Click **Create**

Set hard disk file type.
* Select **VDI (VirtualBox Disk Image)**
* Click **Next**

Set if hard disk space is fixed or allocated as needed.
* Select **Dynamically allocated**
* Select **Next**

Choose a file location and hard drive size for the VM. *Min is 20 GB.  This example uses 60GB.  Reduce or increase this number based on your needs.*
* Change default from 12GB to 60GB.
* Click **Create**

From the VM main menu.
* Select the VM.
* Click **Settings**
* Within Settings, select **Storage**
    * Click on **Controller IDE/Empty** 
    * Use the disk menu to mount **Ubuntu ISO**
* Within Settings, select **Network** *This will allow the VM to connect to the network as if it was an independent machine with it's own virtual MAC*
    * Change **Attached to:** from *NAT* to *Bridged*
    * Click **OK**

**Start** the new VM.

---
## Installing Ubunutu
After bootup, step through the initial Ubuntu setup screens.
* Select language and click **Install Ubunutu**
* Choose a keyboard layout.
* Select **Normal installation**
* Select **Erase disk and install Ubuntu**
* **Continue** to write chages to disk
* Select time zone
* Create name and password. 
    * *For example purposes*
        * Name: SnipeIT
        * Computer: snipeit-VirtualBox
        * Username: snipeit
        * Password: xxxx

---
## Fix Ubuntu desktop scaling in VirtualBox
The Ubunutu GUI will not initially scale correctly with the VirtualBox window.  To fix this we need to install two updates via the terminal.

Bring up the **Terminal** from the bottom left **Show Applications** menu
* Right click and Add to Favorites
* Run the terminal and enter the following:
    ```bash
    sudo apt install virtualbox-guest-dkms
    ```
* From the VM menu, go to **Devices** and select **Insert Guest Additions CD image**.
    * Click **Run** to install the update.
    * Wait for the update to run in the terminal then press any key to exit.
* Restart the VM to replace the running Linux kernal with the updated one.

After the restart the Ubuntu desktop will scale correctly to match the VirtualBox window.

---
## Download and apply system updates
Open the terminal and enter:
```bash    
sudo apt update
```
This will download any available system updates.

Then enter the following to apply them:
```bash
sudo apt upgrade
```
## Configure a static IP address
*This example uses the GUI to configure the static IP address for the server.*

Click the top right control panel to access the settings option.
* Click **Settings** from the drop down.
* Click the **Gear** icon next to the network connection.
    * This example uses IPv4
        * Select **Manual** connection method
        * Update the Address, Netmask, Gateway, and DNS fields with your values.

Restart the VM for the settings to take effect.
* Open the terminal and test the new network settings.
    * Ping the google server.
        ```bash
        ping 8.8.8.8
        ```
    * Once successful, use **Control** + **c** to stop the ping from continuing to run.
    * Enter the following to display the ip configuration.
        ```bash
        ip a
        ```
    * Use the displayed configuration to test pinging your new static IP address from the host machine and another machine on the network.

Alternatively, the CLI can be used to configure the static IP without using the GUI.  
This process can be handled by using Netplan:
[Netplan](https://netplan.io/examples)

---
## Install Apache Server
Use the terminal to download and install the Apache server.
* Open the terminal and run:
```bash
sudo apt install apache2 -y
```
* Once complete you can check the service with the systemctl command
```bash
sudo systemctl status apache2
```

---
## Install PHP
Use the terminal to download and install PHP.
* First, install PHP with the following command:
```bash
sudo apt install php -y
```
* Next, install the needed php extensions.
```bash
sudo apt install php7.4-mbstring php7.4-curl php7.4-mysql php7.4-ldap php7.4-zip php7.4-bcmath php7.4-xml php7.4-gd -y
```

---
## Install MySQL
Use the terminal to download and install MySQL.
* Run the command:
```bash
sudo apt install mysql-server -y
```
* Check the serivce status with systemctl:
```bash
systemctl status mysql
```

---
## Setup the MySQL database
Use the terminal to setup the Snipe-IT database and user in MySQL.
* To start MySQL as the root user enter the following at the command line:
```bash
sudo mysql -u root
```
* Create a database (named snipeitdb in this example) for Snipe-IT.  At the mysql prompt enter:
```SQL
create database snipeitdb;
```
* Create the user account for Snipe-IT. In this example 'snipeituser' is the username and 'snipeit' is their password.
```SQL
create user 'snipeituser'@'localhost' identified by 'snipeit';
```
* Set full rights for the snipeituser on the snipeit database.
```SQL
grant all privileges on snipeitdb.* to 'snipeituser'@'localhost';
```
* Reload the privileges for the new setting to take effect.
```SQL
flush privileges;
```
* Finally, close MySQL.
```SQL
exit;
```

---
## Install the Snipe-IT files
Use the CLI to create the folders needed for Snipe-IT.
* Make the snipeit directory from the command line:
```bash
sudo mkdir /var/www/html/snipeit
```
* Assign ownership of the folder to the snipeit user and group.
```bash
sudo chown snipeit: snipeit /var/www/html/snipeit
```
* Install git to be able to download the latest build of Snipe-IT.
```bash
sudo apt install git
```
* Change directory to the new snipeit folder.
```bash
cd /var/www/html/snipeit
```
* Clone the Snipe-IT repo from git.
```git
git clone https://github.com/snipe/snipe-it .
```

---
## Configure the Snipe-IT files
Make a copy of the snipe config file in the terminal.
* While in the /var/www/html/snipeit/ directory:
```bash
cp .env.example .env
```
* Open the new .env file with the nano editor to update the configuration.
```bash
sudo nano .env
```
* Inside Nano, change the following lines to match your configuration:
    * Under Basic App Settings
    ```
    APP_URL=null to APP_URL=xxx.xx.xx.xxx
    APP_TIMEZONE=’UTC’ to APP_TIMEZONE=’ America/New_York’
    ```
    * Under Database Settings
    ```
    DB_DATABASE=null to DB_DATABASE=snipeitdb
    DB_USERNAME=null to DB_USERNAME=snipeituser
    DB_PASSWORD=null to DB_PASSWORD=snipeit
    ```
    * Under Outgoing Mail Server Setting
    ```
    Setup email configuration here if email functionality wanted.
    ```
    * **Control** + **x** to exit and save the edits.
    * **Y** to save the buffer and enter to overwrite the original file name (.env).

---
## Install the Composer dependency manager for PHP
Install Curl and use it to download the Composer files.
* Run the command:
```bash
sudo apt install curl
```
* Run Curl to get the composer install:
```bash
curl -sS https://getcomposer.org/installer | php
```
* Install Composer using PHP.
```bash
php composer.phar install --no-dev --prefer-source
```

---
## Generate Laravel key for the .env file
Use php to generate the artisan key and populate the APP_KEY line in the .env file. **Note, never share this key.**
* Run command and answer **yes**:
```bash
php artisan key:generate
```

---
## Change ownership of the snipeit directory
Use chown command to give ownership to user:group
* Run command:
```bash
sudo chown -R snipeit:www-data /var/www/html/snipeit
```

---
## Remove write access
Use chmod command to remove write access from group (g-w).
* Run command:
```bash
sudo chmod -R g-w /var/www/html/snipeit
```

---
## Give write access
Use chmod command to give group write access to directory (g+w).
* Run command:
```bash
sudo chmod -R g+w /var/www/html/snipeit/storage
```
* Run command:
```bash
sudo chmod -R g+w /var/www/html/snipeit/public/uploads
```

---
## Setup the Apache website conf file
Copy the 000-default.conf file to create a snipeit.conf file.
* Run command:
```bash
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/snipeit.conf
```
Edit the new snipeit.conf file with nano
* Run command:
```bash
sudo nano /etc/apache2/site-available/snipeit.conf
```
* Replace the text with the following text.  *Update your email and url.*
```
<VirtualHost *:80>
        #Email address of the server manager                
        ServerAdmin webmaster@emailaddeess.domain
        #Public directory for Snipe-IT
        DocumentRoot /var/www/html/snipeit/public
        #APP_URL from the .env file                                           
        ServerName xxx.xx.xx.xxx
        <Directory /var/www/html/snipeit/public>
                Allow From All
                AllowOverride All
                Options -Indexes
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* **Control** + **x** to exit nano.  **Yes** to save.  ENter to overwrite the old file.

---
## Disable the default site
Use the a2dissite to disable the default site.
* Run command:
```bash
sudo a2dissite 000-default.conf
```

---
## Enable the Snipe-IT site
Use the a2ensite command to enable snipeit.conf.
* Run command:
```bash
sudo a2ensite snipeit.conf
```

---
## Enable the rewrite mod
Use the a2enmod command to enable the rewrite mod
* Run command:
```bash
sudo a2enmod rewrite
```

---
## Reload the Apache server
Use the systemctl and reload command to use the updated configuration.
* Run command:
```bash
sudo systemctl reload apache2
```

---
## Login to your Snipe-IT web portal
Snipe-IT LAMP server is now up and running.

Use a web browser to login to your Snipe-IT site and complete the setup process via the Snipe-IT Pre-Flight menu.
