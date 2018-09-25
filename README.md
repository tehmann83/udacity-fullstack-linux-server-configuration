# udacity-fullstack-linux-server-configuration

## Server 

IP address: 18.130.170.135

SSH Port: 2200

URL: [ec2-18-130-170-135.eu-west-2.compute-amazonaws.com](http://ec2-18-130-170-135.eu-west-2.compute.amazonaws.com/)

username/password for udacity reviewer: grader/grader

## Configuration

### Get your server

#### 1. Start a new Ubuntu Linux server instance on Amazon Lightsail

Create instance - OS Only - Ubuntu 16.04 LTS
Name instance: udacity-linux-configuration
Create


#### 2. SSH into server

Go to "Account" -> "SSH Keys" and download the ssh key to your directory /.ssh (name it lightsail-key.pem)
SSH into server with `$ ssh -i ~/.ssh/lightsail-key.pem ubuntu 18.130.170.135`


### Secure your server

#### 3. Update all currently installed packages

`$ sudo apt-get update`
`$ sudo apt-get upgrade`

#### Update automatically

`$ sudo apt-get install unattended-upgrades`
`$ sudo dpkg-reconfigure --priority=low unattended-upgrades`

[Ubuntu - AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates)


#### 4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it

Open `$ sudo nano /etc/ssh/sshd_config` and 
- change `Port` to `2200`,
- change `PermitRootLogin` to `no`,
- change `PasswordAuthentication` to `no`,
- save & exit.


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

`$ ssh-keygen` -> name it `grader_key`

Print key to terminal:

`$ cat grader_key.pub` and COPY it.

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

Now login as grader is possible, on local machine type:

`$ sudo ssh -i ~/.ssh/grader_key grader@18.130.170.135 -p 2200`


#### 9. Configure the local timezone to UTC.

`$ sudo dpkg-reconfigure tzdata` -> "none of the above" -> "utc"


#### 10. Install and configure Apache to serve a Python mod_wsgi application

`$ sudo apt-get install apache2`

Visit `18.130.76.28` to check if Apache2 Ubuntu Default Page is showing up.

`$ sudo apt-get install libapache2-mod-wsgi python-setuptools`

Enable wsgi: `$ sudo a2enmod wsgi`

`$ sudo service apache2 restart`


#### 11. Install and configure PostgreSQL

`$ sudo apt-get install postgresql python-psycopg2`

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

`# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

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

`$ sudo pip install psycopg2-binary`


Change in `views.py`: 
```
if __name__ == '__main__':
  app.run()
```

Change in `database_setup.py`, in `views.py`, and in `dummy-db.py`:
Exchange `sqlite:///usersCatalog.db` with `postgresql://catalog:password@localhost/catalog`

Create a database: `$ sudo python database_setup.py`
Add data: `$ sudo python dummy-db.py`

#### Create a WSGI file:

In /var/www/catalog/ -> `$ sudo nano catalog.wsgi`

Add text to the file:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog")

from views import app as application
application.secret_key = 'secret'
```

#### Change path of client secrets:

In views.py change path to absolute path: `/var/www/catalog/catalog/client_secrets.json`


#### Create Apache config file:

`$ sudo nano /etc/apache2/sites-available/catalog.conf`

Add text:

```
<VirtualHost *:80>
  ServerName 18.130.170.135
  ServerAdmin admin@18.130.170.135
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

Enable host:

`$ sudo a2ensite catalog.conf`

Restart apache:

`$ sudo service apache2 restart`


To make .git directory not publicly accessible, put this in an .htaccess file at the root of your web server.

`RedirectMatch 404 /\.git`


### Errors:

To check for errors on apache server, go to `$ sudo tail /var/log/apache2/error.log`


### Resources:

[http://httpd.apache.org/docs/current/mod/core.html#virtualhost](http://httpd.apache.org/docs/current/mod/core.html#virtualhost/)
[how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
[Deploying python Flask web app on Amazon Lightsail](https://hk.saowen.com/a/0a0048ca7141440d0553425e8df46b16cdf4c13f50df4c5888256393d34bb1b9)
[Make .git directory web inaccessible](https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible)
[Python locale error: unsupported locale setting
](https://stackoverflow.com/questions/14547631/python-locale-error-unsupported-locale-setting)
[How To Install and Use PostgreSQL on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04)
[The pg_hba.conf File](https://www.postgresql.org/docs/9.5/static/auth-pg-hba-conf.html)
[give-all-the-permissions-to-a-user-on-a-db](https://stackoverflow.com/questions/22483555/give-all-the-permissions-to-a-user-on-a-db)


