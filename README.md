# Udacity FSND - Linux server configuration project
This project is for deploying a Flask web application [item catalog website](https://github.com/salam9/catalog) on ubuntu [lightsail AWS](https://aws.amazon.com/) and secure the server.

___
**Public IP Address: http://54.93.211.209/**

**URL: http://ec2-54-93-211-209.eu-central-1.compute.amazonaws.com**

**SSH Port: 2200 Username:grader**
___

## Software Installed On Server
* [Apache2](https://httpd.apache.org)
* [Git](https://git-scm.com)
* [mod_wsgi](https://en.wikipedia.org/wiki/Mod_wsgi)
* [Flask](http://flask.pocoo.org)
* [PostgreSQL](https://www.postgresql.org)
* [Google oAuth](https://developers.google.com/identity/protocols/OAuth2)
* [SQLalchemy](https://www.sqlalchemy.org)
* [pip](https://pypi.org/project/pip/)
* [virtualenv](https://virtualenv.pypa.io/en/latest/)
* [finger](https://linux.die.net/man/1/finger)
* [psycopg](http://initd.org/psycopg/)

## User Management
* Create a new user `$ sudo adduser grader`
* Create a new file under the suoders directory: `$ sudo nano /etc/sudoers.d/grader`. Fill that newly created file with the following line of text: **_grader ALL=(ALL:ALL) ALL_** , then save it.
* Install *finger*, a utility software to check users' status: `$ apt-get install finger`.
* Disable ssh login for *root* user `$ sudo vim /etc/ssh/sshd_config` find the **_PermitRootLogin_** line and edit it to **_no_** then `$ sudo service ssh restart`.

## Security
* Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  ```
  $ sudo ufw status                  # The UFW should be inactive.
  $ sudo ufw default deny incoming   # Deny any incoming traffic.
  $ sudo ufw default allow outgoing  # Enable outgoing traffic.
  $ sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  $ sudo ufw allow www               # Allow HTTP traffic in.
  $ sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  $ sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
  $ sudo ufw enable                  # enable ufw
  ```
 * Changing the SSH port from 22 to 2200`$ sudo vim /etc/ssh/sshd_config` thrn `$ sudo service ssh restart`
* Add rule to Amazon Lightsail firewall Appliction = _custom_ , Protocol = _TCP_ and Portr ange = _2200_
* Now you are able to log into the remote VM through ssh with the following command `$ ssh -i ~/.ssh/lightsail_key.pem -p 2200
ubuntu@54.93.211.209`
*Upgrade installed packages `$ sudo apt-get update` than `$ sudo apt-get upgrade`
## Configure Apache server and deploy the App
* Install Apache `$ sudo apt-get install apache2`
* Install mod_wsgi
  - Run `$ sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `$ sudo a2enmod wsgi`
  - Start the web server with `$ sudo service apache2 start`
* Install git using: `$ sudo apt-get install git`
* Clone my project from github `$ git clone https://github.com/salam9/catalog.git`
* Create a catalog.wsgi file`$ touch catalog.wsgi` , then add this inside:
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'supersecretkey'
  ```
* Rename application.py to __init__.py `$ mv application.py __init__.py`
  
* Install the virtual environment `$ sudo pip install virtualenv`
* Create a new virtual environment with `$ sudo virtualenv venv`
* Activate the virutal environment `$ source venv/bin/activate`
* Change permissions `$ sudo chmod -R 777 venv`
* Install pip with `$ sudo apt-get install python-pip`
* Install Flask `$ pip install Flask`
* Install other project dependencies `$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`
* Run this: `$ sudo vim /etc/apache2/sites-available/catalog.conf`
  - Paste this code: 
  ```
  <VirtualHost *:80>
      ServerName 54.93.211.209
      ServerAlias ec2-54-93-211-209.eu-central-1.compute.amazonaws.com
      ServerAdmin admin@54.93.211.209
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
  ```
* Enable the virtual host `sudo a2ensite catalog`
* Install and configure PostgreSQL
```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres
$ psql
CREATE USER catalog WITH PASSWORD 'password';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
\q
exit
```
  - Change create engine line in your `__init__.py` and `db_setup.py` to: 
  `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
  - `python /var/www/catalog/catalog/db_setup.py`
  
* Restart Apache `$ sudo service apache2 restart`


### Resources & References
- https://github.com/Heba-ahmad/FSND-P5-walkthrough/blob/master/Linux-Server-Configuration-p5-walkthrough.pdf
- https://github.com/rrjoson/udacity-linux-server-configuration/blob/master/README.md
- https://help.ubuntu.com/community/SSH/OpenSSH/Configuring
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04
- https://code.tutsplus.com/tutorials/understanding-virtual-environments-in-python--cms-28272
