# udacity-fullstack-linux-server-configuration

## SSH access to ubuntu instance
Download ssh keys from amazon lightsail and move to your ~/.ssh/ directory
$chmod 400 ~/.ssh/key.pem
Login:
$ssh -i ~/.ssh/key.pm ubuntu@35.178.174.146

## Create a new user

$sudo adduser grader

Give grader sudo permission:
$sudo touch /etc/sudoers.d/grader
$sudo nano /etc/sudoers.d/grader
type in: grader ALL=(ALL:ALL) NOPASSWD:ALL then save&exit

## Create ssh key pair for grader using the ssh-keygen tool
On local machine:
$ssh-keygen
Enter file in which to save the key (~/.ssh/grader)
Enter passphrase twice.
$cat ~/.ssh/grader.pub
Copy public key

On graders vm:
$mkdir .ssh
$touch .ssh/authorized_keys
$nano .ssh/authorized_keys
Paste public key

$chmod 700 .shh
$chmod 644 .ssh/authorized_keys

$sudo nano /etc/ssh/sshd_config  -> check if PasswordAuthentication is set to no

## Secure server
### Updates
$sudo apt-get update
$sudo apt-get upgrade
$sudo apt-get autoremove

### Install finger
$sudo apt-get install finger

### Configure Ubuntu Firewall (UFW)
$sudo ufw status

$sudo ufw default deny incoming
$sudo ufw default allow outgoing

$sudo ufw allow 2200/tcp
$sudo ufw allow 80/tcp
$sudo ufw allow 123/udp

$sudo ufw enable
$sudo ufw status 

## Configure the local timezone to UTC
$sudo dpkg-reconfigure tzdata

### Change ssh port from 22 to 2200
$sudo nano /etc/ssh/sshd_config
Change Port to 2200 on ~line 5

Restart service:
$sudo service ssh restart


##Install and configure Apache to serve a Python mod_wsgi application
Install Apache sudo apt-get install apache2
Install mod_wsgi sudo apt-get install python-setuptools libapache2-mod-wsgi
Restart Apache sudo service apache2 restart

##Install and configure PostgreSQL
Install PostgreSQL sudo apt-get install postgresql

Check if no remote connections are allowed sudo vi /etc/postgresql/9.3/main/pg_hba.conf

Login as user "postgres" sudo su - postgres

Get into postgreSQL shell psql

Create a new database named catalog and create a new user named catalog in postgreSQL shell

postgres=# CREATE DATABASE usersCatalog;
postgres=# CREATE USER usersCatalog;
Set a password for user usersCatalog

postgres=# ALTER ROLE usersCatalog WITH PASSWORD 'usersCatalog';
Give user "usersCatalog" permission to "usersCatalog" application database

postgres=# GRANT ALL PRIVILEGES ON DATABASE usersCatalog TO usersCatalog;
Quit postgreSQL postgres=# \q

Exit from user "postgres"

exit


##Install git, clone and setup your Catalog App project.
Install Git using sudo apt-get install git
Use cd /var/www to move to the /var/www directory
Create the application directory sudo mkdir FlaskApp
Move inside this directory using cd FlaskApp
Clone the Catalog App to the virtual machine $sudo git clone https://github.com/tehmann83/udacity-fullstack-item-catalog
Move to the inner FlaskApp directory using cd FlaskApp

Rename server.py to __init__.py using sudo mv website.py __init__.py, if __init__.py not present.
Edit database_setup.py and fill_catalog.py to change engine = create_engine('sqlite:///catalog.db') to engine = create_engine('postgresql://usersCatalog:usersCatalog@localhost/usersCatalog'), if not already done.


14. Install Flask and other dependencies
    $ sudo apt-get install python-pip
    $ sudo pip install Flask
    $ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
    $ sudo pip install requests









