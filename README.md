# Linux-Server-Configuration

This is the final project for "Full Stack Web Developer Nanodegree" on Udacity.
In this project, took a baseline installation of a Linux distribution on Amazon lightsail and prepared it to host a flask web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

- xip.io url: `http://www.35.182.15.26.xip.io`

* Public ip: `35.182.15.26`

## SSH into AWS server

- Download Private Key from the SSH keys section in the Account section on Amazon Lightsail, for example, to Download file on local computer
- Move the private key file into the folder ~/.ssh: <br />
  `mv ~/Downloads/Lightsail-key.pem ~/.ssh/`

- Change the permission of the file: <br />
  `chmod 400 ~/.ssh/Lightsail-key.pem`

- Remontely login via the local terminal to Amazon Lightsail: <br />
  `ssh ubuntu@35.182.15.26 -i ~/.ssh/LightsailDefaultKey-ca-central-1.pem`

## Create a new user grader on linux

- create a new user grader: `sudo adduser grader`
- give the sudo permission to grader: `sudo nano /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) NOPASSWD:ALL`

## Set ssh login for grader user using private eys

- Generate keys on local machine using `ssh-keygen`, hen save the private key in `~/.ssh` on local machine

* copy the public key that generated to the virtual machine

```
su - grader
mkdir .ssh
touch .ssh/authorized_keys
```

copy the public key generated on local machine(with .pub extension) to .ssh/authorized_keys on virtual machine

- reload SSH using: `sudo service ssh restart`

- modify the permission:

```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```

After completing all these steps, you can login with the new user you created <br />
`ssh grader@35.182.15.26 -i ~/.ssh/[privateKeyFilename]`
then type the password `grader`

## Update all currently installed packages

```
sudo apt-get update
sudo apt-get upgrade
```

## Change the SSH port from 22 to 2200

- Changing port from 22 to 2200 in file `/etc/ssh/sshd_config`

* Changing `PermitRootLogin no`, to disable root login remotely

- Reload SSH using `sudo service ssh restart`

Note: for aws LightSail, in lightsail management consol, add a port for custom application <br />
`Application = Custom, Protocol = TCP, Port = 2200`

## Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
```

## Configure the local timezone to UTC

`sudo dpkg-reconfigure tzdata`, scroll to the bottom of the Continents list and select Etc or None of the above; in the second list, select UTC

## Install and configure PostgreSQL

1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo cat /etc/postgresql/9.3/main/pg_hba.conf`
3. Login in as user postgres by `sudo su - postgres`
4. Get into PostgreSQL by `psql`
5. Create a new database named catalog and create a new user named catalog in postgreSQL shell<br />
   `CREATE USER catalog WITH PASSWORD 'password';` <br />
   `CREATE DATABASE catalog WITH OWNER catalog;`

6. Give user "catalog" permission to "catalog" application database <br />
   `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

7. Exit from user "postgres": `\q`, then `exit`

## Setup for flask app

### Setup python virtual enviroment

- Install virtualenv: sudo pip install virtualenv: `sudo pip3 install virtualenv` <br />

* Set enviornment name using : `sudo virtualenv venv`

* activating the virtual environment using: `source venv/bin/activate`

* install all the packges:

```
sudo apt-get install python3-pip
sudo pip3 install --upgrade pip
pip3 install flask packaging oauth2client redis passlib flask-httpauth
pip3 install sqlalchemy flask-sqlalchemy psycopg2-binary bleach requests
```

Note: in virtual environment, do not use sudo to install package, `sudo pip3 install` will install the package to global environment

### Install and configure Apache to serve a Python mod_wsgi application

```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3 python-dev
sudo service apache2 restart
```

### Install git

```
sudo apt-get install git
```

### Clone and setup catalog app

1. Change the directory to the /var/www directory `cd /var/www`
2. Create the application directory `sudo mkdir catalog`
3. Change the to FlaskApp `cd catalog`
4. Clone the project catalog `sudo git clone https://github.com/lljchem8/Item-Catalog-fullstack.git`
5. Rename the project name `sudo mv ./Item-Catalog-fullstack/ ./catalog/`
6. Move to the inner FlaskApp directory using `cd catalog`
7. Rename `project.py` to `__init__.py` using `sudo mv project.py __init__.py`
8. Edit `database_setup.py`, `lots_of_items.py` and `__init__.py`, change `engine = create_engine('sqlite:///catalog.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`

### Create dummy database

```
python3 database_setup.py
python3 lots_of_items.py
```

Note: change the table name user to users in database_setup.py

### Configure and Enable a New Virtual Host

- Create a file: `sudo nano /etc/apache2/sites-available/catalog.conf`
- Write the following code into the catalog.conf file(`35.182.15.26.xip.io` is the [xip.io](http://xip.io/) address):

```
<VirtualHost *:80>
    ServerName 35.182.15.26
    ServerAlias 35.182.15.26.xip.io
    ServerAdmin lljchem8@gmail.com
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

- Enable it using command: `sudo a2ensite catalog`

### Create the .wsgi File

- Apache uses the .wsgi file to serve the Flask app. Move to the /var/www/catalog directory and create a file named catalog.wsgi with following commands:

```
cd /var/www/catalog
sudo nano catalog.wsgi
```

Add the following lines of code to the catalog.wsgi file, the fisrt three lines are for python3 virtual environment:

```
activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

#!/usr/bin/python3
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from catalog import app as application
application.secret_key = 'Add your secret key'
```

### Oauth Login

- Add http://35.182.15.26 and http://www.35.182.15.26.xip.io to JavaScript origins
- Add http://www.35.182.15.26.xip.io/login and http://www.35.182.15.26.xip.io/gconnect to redirect URIs
- Download the new JSON file of this new setup

Note: Google Oauth does not support public address for redirect URI

### Directory structure

<pre>
| -------- catalog
    |---------------- catalog
        |-----------------------static
        |-----------------------templates
        |-----------------------ven
        |-----------------------__init__.py
    |----------------catalog.wsgi
</pre>

### Restart Apache

Restart Apache with the following command to apply the changes:
`sudo service apache2 restart`

Now you should be able to run the application at [homepage](http://www.35.182.15.26.xip.io/)

### Debugging

If you are getting an Internal Server Error or any other error(s), make sure to check out Apache's error log for debugging:<br/>
`sudo cat /var/log/apache2/error.log`

## Reference

[1] https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps <br />
[2] http://terokarvinen.com/2016/deploy-flask-python3-on-apache2-ubuntu
