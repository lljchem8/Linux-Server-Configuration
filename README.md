# Linux-Server-Configuration

This is the final project for "Full Stack Web Developer Nanodegree" on Udacity.
In this project, took a baseline installation of a Linux distribution on a virtual machine and prepared it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## SSH into AWS server

- Download Private Key from the SSH keys section in the Account section on Amazon Lightsail, for example, to Download file on local computer
- Move the private key file into the folder ~/.ssh:

  `mv ~/Downloads/Lightsail-key.pem ~/.ssh/`

- Change the permission of the file:

  `chmod 400 ~/.ssh/Lightsail-key.pem`

- Remontely login via the local terminal to Amazon Lightsail:

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

After completing all these steps, you can login with the new user you created

`ssh grader@35.182.15.26 -i ~/.ssh/[privateKeyFilename]`
then type the password `grader`

## Update all currently installed packages

```
sudo apt-get update
sudo apt-get upgrade
```

## Change the SSH port from 22 to 2200

- Changing port from 22 to 2200 in file `/etc/ssh/sshd_config`
- Reload SSH using `sudo service ssh restart`

Note: for aws LightSail, in lightsail management consol, add a port for custom application

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

## Install and configure Apache to serve a Python mod_wsgi application

1. Install Apache `sudo apt-get install apache2`
2. install the Python 3 mod_wsgi package `sudo apt-get install libapache2-mod-wsgi-py3`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL

1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo cat /etc/postgresql/9.3/main/pg_hba.conf`
3. Login in as user postgres by `sudo su - postgres`
4. Get into PostgreSQL by `psql`
5. Create a new database named catalog and create a new user named catalog in postgreSQL shell

```
CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog WITH OWNER catalog;
```

6. Give user "catalog" permission to "catalog" application database

   `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

7. Exit from user "postgres": `\q`, then `exit`
