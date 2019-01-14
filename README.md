#  Linux server configuration
## the site IP is : 104.211.18.214 and the site name: http://item-catalog.eastus.cloudapp.azure.com/
### to Enter this site you should visit http://item-catalog.eastus.cloudapp.azure.com/ not the IP. 
#### #what I am using 
in this site I use Microsoft Azure and ubuntu server.
## steps:-
### In the beginning update and upgrader your system then you should download all the dependecnies:
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi
sudo service apache2 start
sudo apt-get install git
sudo apt-get install python-pip
sudo pip install virtualenv
### then create new user name grader 
sudo adduser grader 
password = udacity
#### give the grader sudo prevelagie:
sudo vi /etc/sudoers.d/grader
press i to insert and write: grader ALL=(ALL:ALL) NOPASSWD:ALL 
press esc and then then write :wq to save.
### change ssh port from 22 to 2200 
sudo vi /etc/ssh/sshd_config
change 22 to 2200 and save the file.
sudo service ssh restart
### ssh to grader by doing 
su - grader
mkdir .ssh
touch .ssh/authorized_keys
vi .ssh/authorized_keys and put your public key and save the file.
### allow incoming connection: 
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable
### change timeaone to UTC by writing: 
sudo dpkg-reconfigure tzdata
press enter twice. 
### now you can enter the site using 2200 port 
ssh -p 2200 grader@item-catalog.eastus.cloudapp.azure.com
### running your app:
first clone your app:
cd /var/www
sudo mkdir catalog
Change owner of the newly created catalog folder sudo chown -R grader:grader catalog
cd /catalog
Clone your project from github git clone https://github.com/Hisham-Developer/item_catalog.git catalog
Create a catalog.wsgi file, with this content:
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import app as application
application.secret_key = 'super_secret_key'
### Install virtual environment
Create a new virtual environment with sudo virtualenv venv
Activate it with
source venv/bin/activate
Change permissions sudo chmod -R 777 venv
pip install Flask
pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
#### Create a new file with this : 
sudo vi /etc/apache2/sites-available/000-default.conf
<VirtualHost *:80>
	ServerName 104.211.18.214
	ServerAlias item-catalog.eastus.cloudapp.azure.com
	ServerAdmin webmaster@104.211.18.214
	DocumentRoot /var/www/html
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
	CustomLog ${APACHE_LOG_DIR}/access.log combined
	WSGIScriptAlias / /var/www/catalog/catalog.wsgi
</VirtualHost>
 sudo a2ensite 000-default.conf
 #### set up postgress 
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
CREATE USER catalog WITH PASSWORD 'password';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
##### run:
python /var/www/catalog/catalog/DB.py
restart apache2 and then visit the link:
sudo service apache2 restart

visit: 
http://item-catalog.eastus.cloudapp.azure.com/
