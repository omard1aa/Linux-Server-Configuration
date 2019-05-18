# Linux Server Configuration Project

### About the project
> A baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers


* IP Address: [http://35.159.20.32]()
* URL using DNS: [http://35.159.20.32/](http://35.159.20.32/)
* SSH Port: 22


### Steps Followed to Configure the server
If you speak Arabic, you should watch this video [Discuss Linux Server Configuration Project](https://youtu.be/v9VvJvTyuH0)
#### 1. Update all packages
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```
Enable automatic security updates
```
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

#### 2. Change timezone to UTC and Fix language issues 
```
sudo timedatectl set-timezone UTC

sudo update-locale LANG=en_US.utf8 LANGUAGE=en_US.utf8 LC_ALL=en_US.utf8

```

#### 3. Create a new user grader and Give him `sudo` access
```
sudo adduser grader

sudo nano /etc/sudoers.d/grader 
```
Then add the following text in the file `grader ALL=(ALL) ALL` 
Then `CTRL + x`, press `y` and `enter`

#### 4. Setup SSH keys for grader
* On local machine 
`ssh-keygen`
Then choose the path for storing public and private keys
* On remote machine home as user grader
```
sudo su - grader

mkdir .ssh

touch .ssh/authorized_keys 

sudo chmod 700 .ssh

sudo chmod 644 .ssh/authorized_keys 

nano .ssh/authorized_keys 
```
Then paste the contents of the public key created on the local machine

#### 5. Change the SSH port to 2200 | Enforce key-based authentication | Disable login for root user
```
sudo nano /etc/ssh/sshd_config
```
Then change the following:
* Find the Port line and change 22 to 2200
* Find the PasswordAuthentication line and edit it to no.
* Find the PermitRootLogin line and edit it to no.
* Save the file and run `sudo service ssh restart`

#### 6. Configure the Uncomplicated Firewall (UFW)
```
sudo ufw default deny incoming

sudo ufw default allow outgoing

sudo ufw allow 2200/tcp # --Do not forget to create custom tcp port [2200] on your instance network tab Firewall section 

sudo ufw allow www

sudo ufw allow ntp

sudo ufw enable

sudo service ssh restart
```

#### 8. Install Apache2 and mod-wsgi for python2 and Git
```
sudo apt-get install python-pip

sudo apt-get install python-flask

sudo apt-get install apache2

sudo apt-get install apache2 libapache2-mod-wsgi
```

#### 9. Install and configure PostgreSQL
```
sudo apt-get install libpq-dev python3-dev

sudo apt-get install postgresql postgresql-contrib

sudo su - postgres

psql
```
Then
```
CREATE USER catalog WITH PASSWORD 'password';

CREATE DATABASE catalog WITH OWNER catalog;

\c catalog

REVOKE ALL ON SCHEMA public FROM public;

GRANT ALL ON SCHEMA public TO catalog;

\q

exit
```
**Note:** In your catalog project you should change database engine to
```
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```

#### 10. Clone the Catalog app from GitHub and Configure it
```
cd /var/www/

sudo mkdir catalog

sudo chown grader:grader catalog

git clone <your_repo_url> catalog

cd catalog

git checkout production # If you have a diffrent branch!

nano catalog.wsgi
```
Then add the following in `catalog.wsgi` file
```
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
application.secret_key = 'Super_secret_key'
```
```
sudo apt-get install python-pip
sudo -H pip install virtualenv
virtualenv env
source env/bin/activate
pip install -r requirements.txt
pip install flask packaging oauth2client redis passlib flask-httpauth
pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests
```



#### 11. Configure apache server

sudo nano /etc/apache2/sites-enabled/000-default.conf

Then add the following content:
```
# serve catalog app
<VirtualHost *:80>
  ServerName 35.159.20.32 
  ServerAdmin Omardiaa27@gmail.com
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
```

#### 12. Reload & Restart Apache Server
```
sudo service apache2 reload
sudo service apache2 restart
```


### Resources
* [Flask mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
* [Apache Server Configuration Files](https://httpd.apache.org/docs/current/configuring.html)
* [Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Set Up Apache Virtual Hosts on Ubuntu ](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
* [mod_wsgi documentation](https://modwsgi.readthedocs.io/en/develop/)
* [Automatic Security Updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates#Using_the_.22unattended-upgrades.22_package)
* [Protect SSH with Fail2Ban](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)
* [UFW with Fail2ban](https://askubuntu.com/questions/54771/potential-ufw-and-fail2ban-conflicts)
* [Fix locale issue](https://askubuntu.com/questions/162391/how-do-i-fix-my-locale-issue)
* [Ask Ubuntu](https://askubuntu.com/)
* [Stack Overflow](https://stackoverflow.com/)
