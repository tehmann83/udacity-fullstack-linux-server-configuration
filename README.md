# udacity-fullstack-linux-server-configuration

##Server 

IP address: 35.176.144.139
SSH Port: 2200

username/password for udacity reviewer: grader/grader

##Configuration

1. Start a new Ubuntu Linux server instance on Amazon Lightsail

Download private key LightsailDefaultKey-eu-west-2.pem and move it to .ssh

2. SSH into server

`$ sudo ssh -i ~/.ssh/LightsailDefaultKeyPair.pem ubuntu@35.176.144.139`

3. Update all currently installed packages

`$ sudo apt-get update`
`$ sudo apt-get upgrade`

4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it

`$ sudo nano /etc/ssh/sshd_config`
Change `PasswordAuthentication` to no
Change Port to `2200`
`$ sudo service ssh restart`
Add tcp 2200 rule to aws lightsail

5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

`$ sudo ufw default deny incoming`

`$ sudo ufw default allow outgoing`

`$ sudo ufw allow 2200/tcp`

`$ sudo ufw allow 80/tcp`

`$ sudo ufw allow 123/udp`

`$ sudo ufw enable`

Add tcp 80 and udp 123 to aws lightsail

6. Create a new user account named grader.

`$ sudo adduser grader`

7. Give grader the permission to sudo

8. Create an SSH key pair for grader using the ssh-keygen tool

On locale machine run:

`ssh-keygen`

Enter a file name and save key to `~/.ssh`

Copy the key from `cat ~/.ssh/grader_key.pub`

Login to vm as user grader

On graders vm:

Create directory `sudo mkdir .ssh`

`sudo nano ~/.ssh/authorized_keys`

Paste key from local machine, save, and exit

Change permissions:

`sudo chmod 700 .ssh`

`sudo chmod 644 .ssh/authorized_keys`

`sudo chown grader:grader .ssh`

`sudo chown grader:grader .ssh/authorized_keys`

Set PasswordAuthentication to no:

`sudo nano /etc/ssh/sshd_config`

`sudo service ssh restart`

Login as grader from locale machine: 
`ssh -i ~/.ssh/grader_key grader@35.178.249.129 -p 2200`

9. Configure the local timezone to UTC.

`$ sudo dpkg-reconfigure tzdata`

10. Install and configure Apache to serve a Python mod_wsgi application

11. Install and configure PostgreSQL

12. Install git



