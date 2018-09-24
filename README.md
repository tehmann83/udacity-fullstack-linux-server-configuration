# udacity-fullstack-linux-server-configuration

##Server 

IP address: 
SSH Port: 2200

username/password for udacity reviewer: grader/grader

##Configuration

###Get your server

1. Start a new Ubuntu Linux server instance on Amazon Lightsail

Create instance - OS Only - Ubuntu 16.04 LTS
Name instance: udacity-linux-configuration
Create


2. SSH into server

Go to "Account" -> "SSH Keys" and download the ssh key to your directory /.ssh (name it lightsail-key.pem)
SSH into server with `$ ssh -i ~/.ssh/lightsail-key.pem ubuntu 18.130.76.28`


###Secure your server

3. Update all currently installed packages

`$ sudo apt-get update`
`$ sudo apt-get upgrade`


4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it

Open `$ sudo nano /etc/ssh/sshd_config` and change Port to 2200, save & exit.

`$ sudo service ssh restart`

In Lightsail open "Manage" -> "Networking", and add rule Custom-TCP-2200, save.


5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

`$ sudo ufw status` (ufw should be inactive)

`$ sudo ufw default deny incoming`

`$ sudo ufw default allow outgoing`

`$ sudo ufw allow 2200/tcp`

`$ sudo ufw allow 80/tcp`

`$ sudo ufw allow 123/udp`

`$ sudo ufw enable`

Check if firewall is running and all settings are correct:

`$ sudo ufw status`

In Lightsail open "Manage" -> "Networking", and add rule Custom-TCP-80 (should already be there), and Custom-UPD-123, save.

`$ exit` ssh connection, and ssh into server with newly created port 2200:

`$ ssh -i ~/.ssh/lightsail-key.pem -p 2200 ubuntu@18.130.76.28`


6. Create a new user account named grader.

`$ sudo adduser grader`

Password = `grader`

`$ sudo apt-get install finger`

Check if grader info exists: `finger grader`


7. Give grader the permission to sudo

`$ sudo cat /etc/sudoers` (shows sudoers file)

`$ sudo ls /etc/sudoers.d` (lists sudoers)

`$ sudo nano /etc/sudores.d/grader` to edit sudoers file

Add this line to file: `grader ALL=(ALL:ALL) ALL`, save & quit.

Check if grader is in sudoers:

`$ sudo ls /etc/sudoers.d` (lists sudoers)


8. Create an SSH key pair for grader using the ssh-keygen tool

On local machine:

`$ ssh-keygen` -> name it `grader-key`

Print key to terminal:

`$ cat grader-key.pub` and COPY it.

On VM:

Change user to grader: `$ su - grader`

Create .ssh directory: `$ sudo mkdir .ssh`

`$ sudo nano ~/.ssh/authorized_keys` -> PASTE grader-key.pub content, save & quit

Change permissions:

`$ sudo chmod 700 .ssh`

`$ sudo chmod 644 .ssh/authorized_keys`

Forcing key based authentication:

`$ sudo nano /etc/ssh/sshd_config` -> Set `PasswordAuthentication` to `no`

Restart ssh: `$ sudo service ssh restart`


9. Configure the local timezone to UTC.

`$ sudo dpkg-reconfigure tzdata` -> "none of the above" -> "utc"


10. Install and configure Apache to serve a Python mod_wsgi application

`$ sudo apt-get install apache2`

Visit `18.130.76.28` to check if Apache2 Ubuntu Default Page is showing up.

`$ sudo apt-get install libapache2-mod-wsgi`

Enable wsgi: `$ sudo a2enmod wsgi`


11. Install and configure PostgreSQL

`$ sudo apt-get install postgresql`

Remote connections should not be allowed:

Check `$ sudo cat /etc/postgresql/9.5/main/pg_hba.conf`. Near the bottom you should find these lines:

```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```


12. Install git

13.
14.

