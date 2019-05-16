# Linux-Server-Configuration
About the project
A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers

IP Address:
SSH Port:
URL using DNS:
Steps Followed to Configure the server
If you speak Arabic, you should watch this video Discuss Linux Server Configuration Project

1. Update all packages
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
Enable automatic security updates

sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
2. Change timezone to UTC and Fix language issues
sudo timedatectl set-timezone UTC
sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8
3. Create a new user grader and Give him sudo access
sudo adduser grader
sudo nano /etc/sudoers.d/grader 
Then add the following text grader ALL=(ALL) ALL

4. Setup SSH keys for grader
On local machine ssh-keygen Then choose the path for storing public and private keys
On remote machine home as user grader
sudo su - grader
mkdir .ssh
touch .ssh/authorized_keys 
sudo chmod 700 .ssh
sudo chmod 644 .ssh/authorized_keys 
nano .ssh/authorized_keys 
Then paste the contents of the public key created on the local machine

sudo nano /etc/ssh/sshd_config
Then change the following:

Find the PasswordAuthentication line and edit it to no.
Find the PermitRootLogin line and edit it to no.
Save the file and run sudo service ssh restart
6. Configure the Uncomplicated Firewall (UFW)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 8000/tcp  `serve another app on the server`
sudo ufw enable

8. Install Apache2 and mod-wsgi for python2 and Git
sudo apt-get install apache2 libapache2-mod-wsgi-py

9. Install and configure PostgreSQL
sudo apt-get install libpq-dev python3-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
Then

CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
Note: In your catalog project you should change database engine to

engine = create_engine('postgresql://catalog:password@localhost/catalog')
10. Clone the Catalog app from GitHub and Configure it
cd /var/www/
sudo mkdir catalog
sudo chown grader:grader catalog
git clone <your_repo_url> catalog
cd catalog
git checkout production # If you have a diffrent branch!
nano catalog.wsgi
Then add the following in catalog.wsgi file

#!/usr/bin/python
import sys
sys.stdout = sys.stderr

# -------
activate_this = '/var/www/catalog/env/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))
# -------

sys.path.insert(0,"/var/www/catalog")

from app import app as application
Optional but recommended: Setup virtual environment and Install app dependencies

sudo apt-get install python-pip
sudo -H pip install virtualenv
virtualenv env
source env/bin/activate
pip3 install -r requirements.txt
If you don't have requirements.txt file, you can use
pip3 install flask packaging oauth2client redis passlib flask-httpauth
pip3 install sqlalchemy flask-sqlalchemy psycopg2 bleach requests
Edit Authorized JavaScript origins

12. Configure apache server
sudo nano /etc/apache2/sites-enabled/000-default.conf
Then add the following content:

# serve catalog app
<VirtualHost *:80>
  ServerName 192.168.1.1 // ip addr of vagrant 10.0.2.15 or 10.0.2.2
  ServerAlias <DNS>
  ServerAdmin <Email>
  DocumentRoot /var/www/catalog
  WSGIDaemonProcess catalog user=grader group=grader
  WSGIScriptAlias / /var/www/catalog/catalog.wsgi

  <Directory /var/www/catalog>
    WSGIProcessGroup catalog
    WSGIApplicationGroup %{GLOBAL}
    Require all granted
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


14. Reload & Restart Apache Server
sudo service apache2 reload
sudo service apache2 restart
Resources
Flask mod_wsgi (Apache)
Apache Server Configuration Files
Deploy a Flask Application on an Ubuntu VPS
Set Up Apache Virtual Hosts on Ubuntu
mod_wsgi documentation
Automatic Security Updates
UFW with Fail2ban
Fix locale issue
Ask Ubuntu
Stack Overflow
