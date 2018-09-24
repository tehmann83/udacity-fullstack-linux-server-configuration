# udacity-fullstack-linux-server-configuration

## Server 

IP address: 
SSH Port: 2200

username/password for udacity reviewer: grader/grader

## Configuration

### Get your server

#### 1. Start a new Ubuntu Linux server instance on Amazon Lightsail

Create instance - OS Only - Ubuntu 16.04 LTS
Name instance: udacity-linux-configuration
Create


#### 2. SSH into server

Go to "Account" -> "SSH Keys" and download the ssh key to your directory /.ssh (name it lightsail-key.pem)
SSH into server with `$ ssh -i ~/.ssh/lightsail-key.pem ubuntu 18.130.76.28`


### Secure your server

#### 3. Update all currently installed packages

`$ sudo apt-get update`
`$ sudo apt-get upgrade`


#### 4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it

Open `$ sudo nano /etc/ssh/sshd_config` and change Port to 2200, save & exit.

`$ sudo service ssh restart`

In Lightsail open "Manage" -> "Networking", and add rule Custom-TCP-2200, save.


#### 5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

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


#### 6. Create a new user account named grader.

`$ sudo adduser grader`

Password = `grader`

`$ sudo apt-get install finger`

Check if grader info exists: `finger grader`


#### 7. Give grader the permission to sudo

`$ sudo cat /etc/sudoers` (shows sudoers file)

`$ sudo ls /etc/sudoers.d` (lists sudoers)

`$ sudo nano /etc/sudores.d/grader` to edit sudoers file

Add this line to file: `grader ALL=(ALL:ALL) ALL`, save & quit.

Check if grader is in sudoers:

`$ sudo ls /etc/sudoers.d` (lists sudoers)


#### 8. Create an SSH key pair for grader using the ssh-keygen tool

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


#### 9. Configure the local timezone to UTC.

`$ sudo dpkg-reconfigure tzdata` -> "none of the above" -> "utc"


#### 10. Install and configure Apache to serve a Python mod_wsgi application

`$ sudo apt-get install apache2`

Visit `18.130.76.28` to check if Apache2 Ubuntu Default Page is showing up.

`$ sudo apt-get install libapache2-mod-wsgi`

Enable wsgi: `$ sudo a2enmod wsgi`


#### 11. Install and configure PostgreSQL

`$ sudo apt-get install postgresql`

Remote connections should not be allowed:

Check `$ sudo cat /etc/postgresql/9.5/main/pg_hba.conf`. Near the bottom you should find these lines:

```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

Create a new database user named catalog that has limited permissions to your catalog application database:

Login to psql: 

`$ sudo su - postgres`

`$ psql`

Create database user "catalog":

`# CREATE USER catalog WITH PASSWORD 'password';`

Give user catalog CREATEDB permission:

`# ALTER USER catalog CREATEDB;`

`# CREATE DATABASE catalog;`

Give user catalog permissions on database catalog:

`# GRANT ALL ON DATABASE catalog TO catalog;`

`# \q` to close psql

`$ exit` to return to user `grader`


#### 12. Install git

`$ sudo apt-get install git`


#### 13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program

`$ cd /var/www/`

`$ mkdir catalog` and `cd` into it.

Clone your repository:

`$ sudo git clone https://github.com/tehmann83/udacity-fullstack-catalog-project catalog`


#### 14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!

From `/var/www/catalog/catalog`:

`$ sudo apt-get install python-pip`

`$ sudo pip install flask`

`$ sudo pip install sqlalchemy`

`$ sudo pip install httplib2`

`$ sudo pip install requests`

`$ sudo pip install oauth2client`

`$ sudo pip install passlib`

`$ sudo pip install psycopg2`





Create a WSGI file:

In /var/www/catalog/ -> `$ sudo nano catalog.wsgi`

Add text to the file:

```
import sys
sys.path.insert(0, "/var/www/catalog/")

from views import app as application
application.secret_key = 'secret'
```

Create Apache config file:

`$ sudo nano /etc/apache2/sites-available/catalog.conf`

Add text:

```
<VirtualHost *:80>
  ServerName http://ec2-18-130-76-28.eu-west-2.compute.amazonaws.com/
  ServerAlias 18.130.76.28
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
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable host:

`$ sudo a2ensite catalog.conf`

Restart apache:

`$ sudo apache2ctl restart`


To make .git directory not publicly accessible, put this in an .htaccess file at the root of your web server.

`RedirectMatch 404 /\.git`






