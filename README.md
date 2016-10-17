# linux_server
udacity

linux_server_configuration
Udacity - Linux server configuration project
Description

Goals of the project given by our instructors from Udacity:

You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.
The application meant to be deployed is the Item catalog app, previously developed for Project 3.

Useful info

IP address: 52.34.208.247.

Accessible SSH port: 2200.

Application URL: http://ec2-52-34-208-247.us-west-2.compute.amazonaws.com/.

Step by step walkthrough

1 - Create a new user named grader and grant this user sudo permissions.

Log into the remote VM as root user through ssh: $ ssh root@52.34.208.247.
Add a new user called grader: $ sudo adduser grader.
Create a new file under the suoders directory: $ sudo nano /etc/sudoers.d/grader. Fill that newly created file with the following line of text: "grader ALL=(ALL:ALL) ALL", then save it.
In order to prevent the "sudo: unable to resolve host" error, edit the hosts:
$ sudo nano /etc/hosts.
Add the host: 127.0.1.1 ip-10-20-37-65.
2 - Update all currently installed packages

$ sudo apt-get update.
$ sudo apt-get upgrade.
Install finger, a utility software to check users' status: $ apt-get install finger.
3 - Configure the local timezone to UTC

Open time configuration dialog and set it to UTC with: $ sudo dpkg-reconfigure tzdata.
Install ntp daemon ntpd for a better synchronization of the server's time over the network connection: $ sudo apt-get install ntp.
Source: UbuntuTime.

4 - Configure the key-based authentication for grader user

Generate an encryption key on your local machine with: $ ssh-keygen -f ~/.ssh/udacity_key.rsa.
Log into the remote VM as root user through ssh and create the following file: $ touch /home/grader/.ssh/authorized_keys.
Copy the content of the udacity_key.pub file from your local machine to the /home/grader/.ssh/authorized_keys file you just created on the remote VM. Then change some permissions:
$ sudo chmod 700 /home/grader/.ssh.
$ sudo chmod 644 /home/grader/.ssh/authorized_keys.
Finally change the owner from root to grader: $ sudo chown -R grader:grader /home/grader/.ssh.
Now you are able to log into the remote VM through ssh with the following command: $ ssh -i ~/.ssh/udacity_key.rsa grader@52.34.208.247.
5 - Enforce key-based authentication

$ sudo nano /etc/ssh/sshd_config. Find the PasswordAuthentication line and edit it to no.
$ sudo service ssh restart.
6 - Change the SSH port from 22 to 2200

$ sudo nano /etc/ssh/sshd_config. Find the Port line and edit it to 2200.
$ sudo service ssh restart.
Now you are able to log into the remote VM through ssh with the following command: $ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@52.34.208.247.
Source: Ubuntu forums.

7 - Disable ssh login for root user

$ sudo nano /etc/ssh/sshd_config. Find the PermitRootLogin line and edit it to no.
$ sudo service ssh restart.
Source: Askubuntu.

8 - Configure the Uncomplicated Firewall (UFW)

Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

$ sudo ufw allow 2200/tcp.
$ sudo ufw allow 80/tcp.
$ sudo ufw allow 123/udp.
$ sudo ufw enable.
9 - Configure firewall to monitor for repeated unsuccessful login attempts and ban attackers

Install fail2ban in order to mitigate brute force attacks by users and bots alike.

$ sudo apt-get update.
$ sudo apt-get install fail2ban.
We need the sendmail package to send the alerts to the admin user: $ sudo apt-get install sendmail.
Create a file to safely customize the fail2ban functionality: $ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local .
Open the jail.local and edit it: $ sudo nano /etc/fail2ban/jail.local. Set the destemail field to admin user's email address.
Notes: It doesn't make much sense to use fail2ban when the ssh key-based authentication is enforced. Though it is still useful for other things, like smtp/imap-logins.

Sources: DigitalOcean, Reddit.

10 - Configure cron scripts to automatically manage package updates

Install unattended-upgrades if not already installed: $ sudo apt-get install unattended-upgrades.
To enable it, do: $ sudo dpkg-reconfigure --priority=low unattended-upgrades.
11 - Install Apache, mod_wsgi

$ sudo apt-get install apache2.
Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install mod_wsgi with the following command: $ sudo apt-get install libapache2-mod-wsgi python-dev.
Enable mod_wsgi: $ sudo a2enmod wsgi.
$ sudo service apache2 start.
12 - Install Git

$ sudo apt-get install git.
Configure your username: $ git config --global user.name <username>.
Configure your email: $ git config --global user.email <email>.
13 - Clone the Catalog app from Github

$ cd /var/www. Then: $ sudo mkdir catalog.
Change owner for the catalog folder: $ sudo chown -R grader:grader catalog.
Move inside that newly created folder: $ cd /catalog and clone the catalog repository from Github: $ git clone https://github.com/iliketomatoes/catalog.git catalog.
Make a catalog.wsgi file to serve the application over the mod_wsgi. That file should look like this:
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
Some tweaks were needed to deploay the catalog app, so I made a deployment branch which slightly differs from the master. Move inside the repository, $ cd /var/www/catalog/catalog and change branch with: $ git checkout deployment.
Notes: the .git folder will be inaccessible from the web without any particular setting. The only directory that can be listed in the browser will be the static folder: static assets.

14 - Install virtual environment, Flask and the project's dependencies

Install pip, the tool for installing Python packages: $ sudo apt-get install python-pip.
If virtualenv is not installed, use pip to install it using the following command: $ sudo pip install virtualenv.
Move to the catalog folder: $ cd /var/www/catalog. Then create a new virtual environment with the following command: $ sudo virtualenv venv.
Activate the virtual environment: $ source venv/bin/activate.
Change permissions to the virtual environment folder: $ sudo chmod -R 777 venv.
Install Flask: $ pip install Flask.
Install all the other project's dependencies: $ pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2.
Sources: DigitalOcean, Dabapps.

15 - Configure and enable a new virtual host

Create a virtual host conifg file: $ sudo nano /etc/apache2/sites-available/catalog.conf.
Paste in the following lines of code:
<VirtualHost *:80>
    ServerName 52.34.208.247
    ServerAlias ec2-52-34-208-247.us-west-2.compute.amazonaws.com
    ServerAdmin admin@52.34.208.247
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
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
The WSGIDaemonProcess line specifies what Python to use and can save you from a big headache. In this case we are explicitly saying to use the virtual environment and its packages to run the application.

Enable the new virtual host: $ sudo a2ensite catalog.
Source: DigitalOcean.

16 - Install and configure PostgreSQL

Install some necessary Python packages for working with PostgreSQL: $ sudo apt-get install libpq-dev python-dev.
Install PostgreSQL: $ sudo apt-get install postgresql postgresql-contrib.
Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a tusted user who can access the database software. So let's change the user with: $ sudo su - postgres, then connect to the database system with $ psql.
Create a new user called 'catalog' with his password: # CREATE USER catalog WITH PASSWORD 'sillypassword';.
Give catalog user the CREATEDB capability: # ALTER USER catalog CREATEDB;.
Create the 'catalog' database owned by catalog user: # CREATE DATABASE catalog WITH OWNER catalog;.
Connect to the database: # \c catalog.
Revoke all rights: # REVOKE ALL ON SCHEMA public FROM public;.
Lock down the permissions to only let catalog role create tables: # GRANT ALL ON SCHEMA public TO catalog;.
Log out from PostgreSQL: # \q. Then return to the grader user: $ exit.
Inside the Flask application, the database connection is now performed with:
engine = create_engine('postgresql://catalog:sillypassword@localhost/catalog')
Setup the database with: $ python /var/www/catalog/catalog/setup_database.py.
To prevent potential attacks from the outer world we double check that no remote connections to the database are allowed. Open the following file: $ sudo nano /etc/postgresql/9.3/main/pg_hba.conf and edit it, if necessary, to make it look like this:
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
Source: DigitalOcean.

17 - Install system monitor tools

$ sudo apt-get update.
$ sudo apt-get install glances.
To start this system monitor program just type this from the command line: $ glances.
Type $ glances -h to know more about this program's options.
Source: eHowStuff.

18 - Update OAuth authorized JavaScript origins

To let users correctly log-in change the authorized URI to http://ec2-52-34-208-247.us-west-2.compute.amazonaws.com/ on both Google and Facebook developer dashboards.
19 - Restart Apache to launch the app

$ sudo service apache2 restart.
