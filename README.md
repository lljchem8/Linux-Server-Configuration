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

  `ssh ubuntu@35.183.136.251 -i ~/.ssh/LightsailDefaultKey-ca-central-1.pem`

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

`ssh grader@35.183.136.251 -i ~/.ssh/[privateKeyFilename]`

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
