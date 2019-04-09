# Linux-Server-Configuration---Udacity
#####  In this project, a Linux virtual machine needs to be configurated to support the  Catalog App website.
#####  You can visit http://35.173.249.122 for the website deployed.
-- 
##### 1 .Launch your Virtual Machine with your Udacity account (Start a new instance of Ubuntu Linux server on Amazon Lightsail.)
##### 2 .Create a new user named grader
##### 3 .Give the grader the permission to sudo
##### 4 .Update all currently installed packages
##### 5 .Change the SSH port from 22 to 2200
##### 6 .Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
##### 7 .Configure the local timezone to UTC
##### 8 .Install and configure Apache to serve a Python mod_wsgi application
##### 9 .Install and configure PostgreSQL:
- Do not allow remote connections
- Create a new user named catalog that has limited permissions to your catalog application database
##### 10 .Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

# Create a new user named grader
```sh
1 .sudo adduser grader
2 .nano /etc/sudoers
3 .touch /etc/sudoers.d/grader
4 .nano /etc/sudoers.d/grader, type in grader ALL=(ALL:ALL) ALL, save and quit
```
# Set ssh login using keys
1 .generate keys on local machine using ``` ssh-keygen ``` ; then save the private key in ``` ~/.ssh ``` on local machine
2 .deploy public key on developement enviroment

On you virtual machine:
```sh
$ su - grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ vim .ssh/authorized_keys
```
Copy the public key generated on your local machine to this file and save
```sh
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```
3 .reload SSH using ``` service ssh restart ```
4 .now you can use ssh to login with the new user you created

``` ssh -i [privateKeyFilename] grader@35.173.249.122 ```

# Update all currently installed packages
```sh
sudo apt-get update
sudo apt-get upgrade
```
# Change the SSH port from 22 to 2200
1 .Use ``` sudo nano /etc/ssh/sshd_config ``` and then change Port 22 to Port 2200 , save & quit.
2 .Reload SSH using ``` sudo service ssh restart ```

# Configure the Uncomplicated Firewall (UFW)
Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
```sh
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable 
```
# Configure the local timezone to UTC
1 .Configure the time zone ``` sudo dpkg-reconfigure tzdata ```
2 .It is already set to UTC.

# Install and configure Apache to serve a Python mod_wsgi application
1 .Install Apache ``` sudo apt-get install apache2``` 
2 .Install mod_wsgi ``` sudo apt-get install python-setuptools libapache2-mod-wsgi```
3 .Restart Apache ```sudo service apache2 restart```

# Install and configure PostgreSQL
1 .Install PostgreSQL ```sudo apt-get install postgresql```
2 .Check if no remote connections are allowed ```sudo nano /etc/postgresql/9.3/main/pg_hba.conf```
3 .Login as user "postgres" ```sudo su - postgres```
4 .Get into postgreSQL shell ```psql```
5 .Create a new database named catalog and create a new user named catalog in postgreSQL shell
```sh
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
```
6 .Set a password for user catalog
```sh
postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
```
7 .Give user "catalog" permission to "catalog" application database
```sh
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```
8 .Quit postgreSQL ```sh postgres=# \q```

9 .Exit from user ```"postgres"```

exit

# Install git, clone and setup your Catalog App project.
1 .Install Git using ```sudo apt-get install git```
2 .Use ```cd /var/www ```to move to the /var/www directory
3 .Create the application directory ```sudo mkdir FlaskApp```
4 .Move inside this directory using ```cd FlaskApp```
5 .Clone the Catalog App to the virtual machine``` git clone https://github.com/vamdcadm/Catalog-App```
6 .Rename the project's name ```sudo mv ./Catalog-App/FlaskApp```
7 .Move to the inner FlaskApp directory using ```cd FlaskApp```
8 .Rename ```application.py``` to ```__init__.py``` using ```sudo mv application.py __init__.py```
9 .Edit ```db_setup.py```,``` __init__.py``` and ```basic.py``` and change``` engine = create_engine('sqlite:///toyshop.db')``` to ```engine = create_engine('postgresql://catalog:password@localhost/catalog')```
10 .Install pip ```sudo apt-get install python-pip```
11 .Use pip to install dependencies ```sudo pip install -r requirements.txt```
12 .Install psycopg2 ```sudo apt-get -qqy install postgresql python-psycopg2```
13 .Create database schema ```sudo python db_setup.py```

#Configure and Enable a New Virtual Host
1 .Create FlaskApp.conf to edit: ```sudo nano /etc/apache2/sites-available/FlaskApp.conf```
2 .Add the following lines of code to the file to configure the virtual host.
```
<VirtualHost *:80>
	ServerName 35.173.249.122
	ServerAdmin xxxxx@gmail.com
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
3 .Enable the virtual host with the following command: ```sudo a2ensite FlaskApp```

# Create the .wsgi File
1 .Create the .wsgi File under ```/var/www/FlaskApp:```

```cd /var/www/FlaskApp```
```sudo nano flaskapp.wsgi ```
2 .Add the following lines of code to the flaskapp.wsgi file:
```sh
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'Add your secret key'
```
# Restart Apache
1. Restart Apache sudo service apache2 restart

# References:
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://stackoverflow.com/
- http://flask-sqlalchemy.pocoo.org/2.3/
- https://www.postgresql.org/
