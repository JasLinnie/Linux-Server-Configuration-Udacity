# Linux-Server-Configuration-Udacity
This project is deploying the [Item Catalog Application](https://github.com/JasLinnie/udacity-itemcatalog) to be hosted on a Ubuntu Linux server on an Amazon Lightsail instance. To set this up, please go through the following instructions. The application is deployed live on http://13.250.107.163.

## Create a Ubuntu Linux Server instance on Amazon Lightsail
1. Log in to Lightsail or create an AWS account
2. Click **Create an instance** 
3. Select **Linux/Unix** platform
4. Select **OS Only** and **Ubuntu** for blueprint
5. Select an instance plan
6. Name your instance
7. Click **Create** 

## SSH into your Server instance
Within the instance you just created, you can click **Connect using SSH** to SSH into your server instance. Alternative, you can follow these steps to connect using your own SSH client:
1. In your Lightsail account, click **Account** on the top navigation bar and click **Account** again
2. Click **SSH keys** tab
3. Click download to download the private key. It should be a .pem file
4. Create a new .rsa file e.g. lightsail_key.rsa within your local ~/.ssh folder
5. Copy and paste content from downloaded private key file into .rsa file
6. Set file permission as owner only using `$ chmod 600 ~/.ssh/lightsail_key.rsa`
7. SSH into the instance using `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.250.107.163`

## Update all currently installed packages
1. In either the browser-based ssh client launched from Lightsail or your own ssh client, run `sudo apt-get update` to update packages
2. Run `sudo apt-get upgrade` to install the newest versions of the packages
3. Set this for any future updates: `sudo apt-get dist-upgrade`

## Change the SSH port
1. Run `$ sudo nano /etc/ssh/sshd_config` to open up the configuration file
2. Change the port number from **22** to **2200** in this file
3. Save and exit the file
4. Restart SSH using `$ sudo service ssh restart`

## Configure the firewall
1. Check the firewall status using `$ sudo ufw status`
2. Set the default firewall to deny all incomings using `$ sudo ufw default deny incoming`
3. Set the default firewall to allow all outgoings using `$ sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 to allow SSH using `$ sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 to allow www using `$ sudo ufw allow www`
6. Allow incoming UDP packets on port 123 to allow NTP using `$ sudo ufw allow 123/udp`
7. Close port 22 using `$ sudo ufw deny 22`
8. Enable firewall using `$ sudo ufw enable`
9. Check the current firewall status using `$ sudo ufw status`
10. Update the firewall configuration in your Amazon Lightsail instance by going to the **Networking** tab. 
11. Delete default SSH port 22 and add ports with these settings: 
```
      HTTP 80/TCP
      Custom 123/UDP
      Custom 2200/TCP 
```
12. If you are using your own ssh client, open up a new terminal and try ssh in via the new port 2200 using `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.250.107.163 -p 2200`
13. If you ssh in using Lightsail's browser-based ssh client, you can no longer use that. Open up a terminal and try ssh in via the new port 2200 using `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.250.107.163 -p 2200` 

If you are using Windows PC, you can download a third party application to do this. You can download the Putty file as well as Putty key generator file [here](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). After installing them, follow these steps:
1. Open up putty generator application
2. Click **Conversions** and then **Import key**
3. Select the private key .pem file you downloaded from your instance
4. Click **Save Private Key** and save the file
5. Open putty application and enter the 13.250.107.163 IP address and port 2200
6. Connection type should be **SSH**
7. In left tree, navigate to Connection > SSH > Auth
8. Click **Browse** and select the saved file from putty generator 
9. Go back to **Session** and name the session and save
10. Click **Open** to ssh into the instance

## Create user account **grader**
1. Create a new user account **grader** using `$ sudo adduser grader`
2. Create a file using `$ sudo touch /etc/sudoers.d/grader`
3. Edit the file using `$ sudo nano /etc/sudoers.d/grader`
4. Add this `grader ALL=(ALL:ALL) ALL` to the file and save it

## Set SSH login for user grader
1. Open a terminal in your local machine, use `ssh-keygen` to create an SSH key pair for **grader** and save it in the `~/.ssh` path
2. Insert the generated public key into the ubuntu server 
    * In your local machine's `~/.ssh` folder, open and read the generated public key using
     `cat ~/.ssh/FILE-NAME.pub`
    * In your ubuntu server, create the .ssh folder inside the grader folder i.e. /home/grader, and within it create the authorized_keys file:
   ```
      $ mkdir .ssh
      $ touch .ssh/authorized_keys
      $ nano .ssh/authorized_keys
      ```
    * Copy the generated public key and paste it into this authorized_keys file and save it
3. In your terminal logged in to the ubuntu server, use `chown -R grader.grader /home/grader/.ssh` to change ownership and permissions of the .ssh folder to grader 
4. Run `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys` to change the permissions too
5. Restart SSH using `$ sudo service ssh restart`
6. Try to login in as user grader using `$ ssh -i ~/.ssh/grader_key -p 2200 grader@13.250.107.163`
   Or if you are using Putty, follow the same steps provided above to convert the key and SSH into the instance as grader
7. You will be asked for grader's password. To disable it, open the configuration file using `$ sudo nano /etc/ssh/sshd_config`
8. Change `PasswordAuthentication yes` to **no**
9. Restart SSH using `$ sudo service ssh restart`

## Configure the local timezone to UTC
1. Run `$ sudo dpkg-reconfigure tzdata`
2. Choose **None of the above** to set timezone to UTC

## Install and configure Apache
1. Install **Apache** using `$ sudo apt-get install apache2`
2. Go to http://18.218.99.181/, if Apache is working correctly, a **Apache2 Ubuntu Default Page** will show up

## Install and configure Python mod_wsgi
1. Install the **mod_wsgi** package: `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable **mod_wsgi**: `$ sudo a2enmod wsgi`
3. Restart **Apache**: `$ sudo service apache2 restart`
4. Check if Python is installed: `$ python`

## Install PostgreSQL
1. Run `$ sudo apt-get install postgresql`
2. Make sure PostgreSQL does not allow remote connections
3. Open file: `$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
4. Check to make sure it looks like this:
   ```
   # Database administrative login by Unix domain socket
   local   all             postgres                                peer

   # TYPE  DATABASE        USER            ADDRESS                 METHOD

   # "local" is for Unix domain socket connections only
   local   all             all                                     peer
   # IPv4 local connections:
   host    all             all             127.0.0.1/32            md5
   # IPv6 local connections:
   host    all             all             ::1/128                 md5
   ```
## Create new PostgreSQL user called **catalog**
1. Switch to PostgreSQL defualt user **postgres**: `$ sudo su - postgres`
2. Connect to PostgreSQL: `$ psql`
3. Create user **catalog** with LOGIN role: `# CREATE ROLE catalog WITH PASSWORD 'password';`
4. Allow user to create database tables: `# ALTER USER catalog CREATEDB;`
5. Create database: `# CREATE DATABASE catalog WITH OWNER catalog;`
6. Connect to database **catalog**: `# \c catalog`
7. Revoke all the rights: `# REVOKE ALL ON SCHEMA public FROM public;`
8. Grant access to **catalog**: `# GRANT ALL ON SCHEMA public TO catalog;`
9. Exit psql: `\q`
10.Exit user **postgres**: `exit`

## Create new Linux user called **catalog** and new database
1. Create a new Linux user: `$ sudo adduser catalog`
2. Give **catalog** user sudo access:
   * `$ sudo visudo`
   * Add `$ catalog ALL=(ALL:ALL) ALL` under line `$ root ALL=(ALL:ALL) ALL`
   * Save and exit the file
3. Log in as **catalog**: `$ sudo su - catalog`
4. Create database **catalog**: `createdb catalog`
5. Exit user **catalog**: `exit`

## Install git and clone catalog application from github
1. Run `$ sudo apt-get install git`
2. Create dictionary: `$ mkdir /var/www/catalog`
3. CD to this directory: `$ cd /var/www/catalog`
4. Clone the catalog app: `$ sudo git clone RELEVENT-URL catalog`
5. Change the ownership: `$ sudo chown -R ubuntu:ubuntu catalog/`
6. CD to `/var/www/catalog/catalog`
7. Change file **application.py** to **__init__.py**: `$ mv application.py __init__.py`
8. Change line `app.run(host='0.0.0.0', port=8000)` to `app.run()` in **__init__.py** file

## Edit client_secrets.json file
1. Create a new project on Google API Console and download `client_scretes.json` file
2. Copy and paste contents of downloaded `client_scretes.json` to the file with same name under directory `/var/www/catalog/catalog/client_secrets.json`

## Setup for deploying a Flask App on Ubuntu VPS
1. Install pip: `$ sudo apt-get install python-pip`
2. Install packages:
```
   $ sudo pip install httplib2
   $ sudo pip install requests
   $ sudo pip install --upgrade oauth2client
   $ sudo pip install sqlalchemy
   $ sudo pip install flask
   $ sudo apt-get install libpq-dev
   $ sudo pip install psycopg2
   ```

## Setup and enble a virtual host
1. Create file: `$ sudo touch /etc/apache2/sites-available/catalog.conf`
2. Add the following to the file:
```
   <VirtualHost *:80>
		ServerName XX.XX.XX.XX
		ServerAdmin admin@xx.xx.xx.xx
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```
3. Run `$ sudo a2ensite catalog` to enable the virtual host
4. Restart **Apache**: `$ sudo service apache2 reload`

## Configure .wsgi file
1. Create file: `$ sudo touch /var/www/catalog/catalog.wsgi`
2. Add content below to this file and save:
```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/nuevoMexico/")

   from nuevoMexico import app as application
   application.secret_key = 'super_secret_key'
```
3. Restart **Apache**: `$ sudo service apache2 reload`

## Edit the database path
1. Replace lines in `__init__.py`, `database_setup.py`, and `lotsofitems.py` with `engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')`

## Disable defualt Apache page
1. `$ sudo a2dissite 000-defualt.conf`
2. Restart **Apache**: `$ sudo service apache2 reload`

## Set up database schema
1. Run `$ sudo python database_setup.py`
2. Run `$ sudo python lotsofitems.py`
3. Restart **Apache**: `$ sudo service apache2 reload`
4. Now follow the link to http://18.218.99.181/  the application should be runing online
5. If internal errors occur: check the [Apache error file](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)

## Sources
1. [Amazon Lightsail Website](https://aws.amazon.com/lightsail/?p=tile)
2. [Google API Concole](https://console.cloud.google.com/)
3. [Udacity](https://www.udacity.com)
4. [Apache](https://httpd.apache.org/docs/2.2/configuring.html)
