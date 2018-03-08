# Udacity Linux-Server-Configuration
Udacity project of configuring a server with the use of amazonlightsail to host a previous worked project the Item Catalog.

you can visit the deployed webiste here : http://54.159.228.41/

# Change SSH ports from 22 to 2200
- Use `sudo nano /etc/ssh/sshd_config` and changed Port 22 to 2200, save and quit.
- reload ssh using `sudo service ssh restart`
Note: moved Port 22 to a second line during project to use lightsail and gitBash got rid near end.

# Instructions for SSH access to the instance
- Download Private key from the SSH keys in the Account section of Amazon Lightsail.
- Move Private key into folder <User>/.ssh of local computer directory. 
- in gitBash terminal type in `ssh -i ~/.ssh/LightsailDefaultPrivateKey-us-east-1.pem -p 2200 ubuntu@54.159.228.41`

# Configure Firewall (UFW)
- Configured the UFW or Uncomplicated Firewall to allow incoming connections for ssh port 2200 from gitBash, HTTP port 80 for the web and NTP port 123:
```
sudo ufw allow ssh
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable 
sudo ufw status
```
# Update currently installed packages
- Used `sudo apt-get update` then `sudo apt-get upgrade`

# Configure the local timezone to UTC
- configue time zone `sudo dpkg-reconfigure tzdata` to eastern time.

# Creating a new user named grader
- `sudo adduser grader`
- set password conform password (add info if needed)
- `sudo ls /etc/sudors.d`
- `sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader`
- `sudo nano /etc/sudoers.d/grader`, change 90-cloud-init-users to grader, save and quit.

# Set ssh login using keys
- On local Machine (user's computer) use `ssh-keygen` then save private key in `/c/Users/<User>/.ssh/linuxCourse` on local machine
- deploy public key (note: passpharse is asked to add for extra security make it unique if used)
- On virtual machine:
```
  sudo nano grader 
  mkdir .ssh 
  touch .ssh/authorized_keys  
  sudo nano .ssh/authorized_keys
  ```
  
- Copy the public key generated on the local Machine to file and save then:

  `chmod 700 .ssh`
  
  `chomd 644 .ssh/authorized_keys`
  
- Exit grader and now you can log back into grader as:

  ` ssh grader@54.237.161.20 -p 2200 -i ~/.ssh/linuxCourse`

# Install and configure Apache to serve a Python mod_wsgi application
- Install Apache ` sudo apt-get install apache2`
- Install mod_wsgi ` sudo apt-get install libapache2-mod-wsgi`
- restart apache `sudo service apache2 restart`

# Install and configure PostgreSQL
- Install PostgreSQL `sudo apt-get install postgresql`
- login as user "postgres" `sudo su - postgres`
- Get into postgreSQL shell `psql`
- Creare a database named catalog and a new user named catalog in the shell
`CREATE DATABASE catalog;`
AND
`CREATE USER catalog;`
- Set a password for catalog
`ALTER ROLE catalog WITH PASSWORD 'password';`
- Only let user catalog create tables with
`REVOKE ALL ON SCHEMA public FROM public`
then
`GRANT ALL ON SCHEMA public TO catalog;`
- Quit postgreSQL `\q`
- Logout from user "postgres" 

# Install git, clone and setup the Item Catalog project
- Install Git using `sudo apt-get install git`
- Use `cd/var/www` to move to the directory
- Create the application directory `sudo mkdir FlaskApp`
- Move to FlaskApp `cd FlaskApp`
- Clone the Item-Catalog to the VM `sudo git clone https://github.com/Smejia723/Item-Catalog.git`
- Rename the project's name `sudo mv ./Item-Catalog ./FlaskApp`
- Move to the inner FlaskApp directory using `cd FlaskApp`
- Rename `project.py` to `__init__.py` using `sudo mv project.py __init_.py`

- Edit `database_setup.py` and `lotsofmenus.py` to change 
`engine = create_enging('sqlite:///restaurantmenu.db')` to 
`engine = create_enging('postgresql://catalog:password@localhost/catalog')`
- Install pip `sudo apt-get install python-pip`
- Use pip to install dependencies:
```
sudo pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests
sudo pip install flask packaging oauth2client redis passlib flask-httpauth
```
- Install spycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
- Create database `sudo python database_setup.py`
- Fill database `python /var/www/FlaskApp/FlaskApp/lotsofmenus.py`

# Configure and Enable a New Virtual Host
- Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`

- Add the flowing to the file:
```
<VirtualHost *:80>
	ServerName Item_Catalog.py
	ServerAdmin 54.159.228.41
	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
	<Directory /var/www/FlaskApp/FlaskApp/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/FlaskApp/static
	<Directory /var/www/FlaskApp/FlaskApp/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Enable the virtual host with `sudo a2ensite FlaskApp`
- Restart apache `sudo service apache2 restart`

# Create the Catalog.wsgi File
- Create the .wsgi File under 
`cd /var/www/FlaskApp`
then
`sudo nano flaskapp.wsgi`
- Add the flowing to the file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
```
- Restart apache `sudo service apache2 restart`

# References:

http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/#creating-a-wsgi-file

https://discussions.udacity.com/t/configuring-the-firewall-for-amazon-lightsail/222208

https://help.github.com/articles/cloning-a-repository/

https://discussions.udacity.com/t/connection-error-linux/241400/32

https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04

https://discussions.udacity.com/t/connection-error-linux/241400/36

https://discussions.udacity.com/t/installing-virtualenv/224005/2
