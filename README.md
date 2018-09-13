# linux-server-configuration

This project outlines the installation of a Linux server and prepare it to host a web applications, install and configure a database server, and deployment.

*  IP Address: 34.201.54.34
*  SSH Port: 2200
*  URL:  http://34.201.54.34.xip.io

## Resources

* [Amazon Lightsail](https://aws.amazon.com/lightsail/)
* [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Flask](http://flask.pocoo.org/docs/1.0/)
* [SQLAlchemy - PostgreSQL](http://docs.sqlalchemy.org/en/latest/dialects/postgresql.html)
* [Google Oauth 2 Authentication](https://developers.google.com/identity/protocols/OAuth2)
* Facebook Oauth Authentication
	**NOTE:**  Facebook now enforce HTTPS (see [here](https://developers.facebook.com/blog/post/2018/06/08/enforce-https-facebook-login/))


## Setup new Ubuntu Linux server instance on Amazon Lightsail
1.  Go to [Ligthsail](https://aws.amazon.com/lightsail/)
2.  Click Create an AWS Account (yellow button, upper right-hand corner)
3.  Choose an instance image: Ubuntu
4.  Choose your instance plan
5.  Give your instance a hostname
6.  Wait for it to start up
7.  It's running on IPls `t` with the name `linux-catalog`.
8.  Obtain DNS name using the [xip.io](http://xip.io/) service (for OAuth set up). I will use `34.201.54.34.xip.io`. ([set up DNS Zone in Lightsail](https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-create-dns-entry))
9. Connect using SSH button and update all currently installed packages.
	Type `sudo apt-get update`
	Type `sudo apt-get upgrade`

## Create a New User

1. Type `sudo adduser grader`
2. Confirm new user using Finger
	Type `sudo apt-get install finger`
	Type `finger grader`
3. Give `grader` sudo.
	Type `sudo nano /etc/sudoers.d/grader`
	Enter `grader ALL=(ALL) NOPASSWD:ALL` and save file.
 
## Secure the server

### Local Machine

1. On local machine, generate key pairs:
	Type `ssh-keygen`
2. Enter file in which to save the key.
3. Enter passphrase.
4. Private and Public key will be saved on local machine in separte files (NOTE: public file has the same name as the private key, but with .pub on the end)

### Remote Server

1. Log into server and change to `grader`
	Type `su - grader` and enter password at prompt to switch to `grader`
2. In the home directory, add `.ssh` directory
	Type `mkdir .ssh`
3. Create file to store public key
	Type `nano .ssh/authorized_keys`
	Copy public key from local machine and paste into the file and save.
4. Change permission for .ssh directory and ssh key file:
	Type `chmod 700 ~/.ssh`
	Type `chmod 644 ~/.ssh/authorized_keys`
5. Disable password based login
	Type `sudo nano /etc/ssh/sshd_config`
	Find 'Port 22' and change 22 to 2200
	Find 'PasswordAuthentication yes' in the file and 	change 'yes' to ‘no’
	Save and close file.
6. Type `sudo ufw default deny incoming`
7. TYpe `sudo ufw default allow outgoing`
8. Allow ssh by typing `sudo ufw allow ssh`
9. Allow access on port 2200 by typing `sudo ufw allow 2200/tcp`
10. Allow HTTP by typing `sudo ufw allow http`
11. Allow access on port 123 by typing `sudo ufw allow 123/udp`
12. Enable firewall by typing `sudo ufw enable`
13. Restart ssh service
	Type `sudo service ssh restart`
14. In Lightsail (on you server), go to the Networking
tab to update Firewall settings:
	Add Custom TCP 2200
	Add Custom UDP 123
	Save updates.
15. From local machine, verify you can SSH into server using port 2200:
	Type ` ssh grader@34.201.54.34 -p 2200 -i ~/.ssh/linuxProject`
	Enter passphrase for key
	Successfully logged in as `grader` via SSH
16.	In Lightsail (on you server), go to the Networking
tab to edit Firewall settings:
	Remove SSH TCP 22

## Prepare to deploy the project

### Set timezone to UTC

1.  Configure the local timezone to UTC.
	Type `sudo dpkg-reconfigure tzdata`
	Scroll to the bottom of the Continents list and select None of the above
	In the second list, select UTC
	source: [https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
	
### Install Apache

1. Install apache
	Type `sudo apt-get install apache2`
2. Install and Enable mod_wsgi
	Type `sudo apt-get install libapache2-mod-wsgi`
	Type `sudo a2enmod wsgi`
	
### Install Git

1. Install git
	Type `sudo apt-get install git`
	
### Set up application structure

1. Create app
	Type `cd /var/www`
	Type `sudo mkdir CatalogApp`
	Type `cd CatalogApp`
	Clone repository from Catalog Project (completed earlier in course)
		Type `sudo git clone https://github.com/MKing301/catalog.git CatalogApp`
	Rename `catalog_app.py` to `__init__.py`
		Type `cd CatalogApp`
		Type `sudo mv catalog_app.py __init__.py`
2. Make sure that your .git directory is not publicly accessible via a browser (see [here](https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible))
	 
### Install Flask and Set-Up Virtual Environment

1. Install PIP
	Type `sudo apt-get install python-pip`
2. Install virtualenv
	Type `sudo pip install virtualenv`
3. Name virtual environment
	Type `sudo virtualenv venv`
4. Activate vitual environment
	Type `source venv/bin/activate` NOTE: (venv) will appear at begining of line
5. Install Flask here
	Type `sudo pip install Flask`
6. Type `deactivate` to exit virtual environment

### Install additional required packages

1. Install all of the following:
	Type `sudo apt-get install python-psycopg2`
	Type `sudo pip install psycopg2-binary`
	Type `sudo pip install SQLAlchemy`
	Type `sudo pip install Flask-SQLAlchemy`
	Type `sudo pip install requests`
	Type `sudo pip install httplib2`
	Type `sudo pip install flask_httpauth`
	Type `sudo pip install oauth2client`

### Configure and Enable Virtual Host

1. Type `sudo nano /etc/apache2/sites-available/CatalogApp.conf`
2. Edit file to contain the following:
	```
	<VirtualHost *:80>
		ServerName 34.201.54.34.xip.io
		ServerAdmin admin@34.201.54.34.xip.io.com
		WSGIScriptAlias / /var/www/CatalogApp/catalogapp.wsgi
		<Directory /var/www/CatalogApp/CatalogApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /templates /var/www/CatalogApp/CatalogApp/templates
		<Directory /var/www/CatalogApp/CatalogApp/templates/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Save and close file.
4. Create wsgi file.
	Type `cd /var/www/CatalogApp`
	Type `sudo nano catalogapp.wsgi`
	Add the following lines in the file:
		```
		#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0,"/var/www/CatalogApp/")
		from CatalogApp import app as application
		```
5. Disable default:
	Type `sudo a2dissite 000-default.conf`
6. Add ip and host
	Type `sudo nano /etc/hosts`
	Add `34.201.54.34 34.201.54.34.xip.io`
7. Enable virtual host:
	Type `sudo a2ensite CatalogApp`
8. Restart apache
	Type `sudo service apache2 restart`
	
## Install and Set-Up Database

1. Type `apt-get install postgresql` to install
2. Type `sudo -u postgres psql postgres` to access postgreSQL
3. Type `\password postgres` to set the password
4. Type `CREATE DATABASE catalog;` to create the database
5. Type `\c catalog` to connect to the database
6. Create user catalog by typing `Create a user with: postgres=# CREATE USER <username> WITH PASSWORD '<password>';`
7. Grant user catalog permission to database catalog:
	Type `GRANT ALL PRIVILEGES ON DATABASE <database name> TO <username>;`
8. Type `\q` then `ENTER` to exit postgreSQL
9. Reload postgresql by type `sudo /etc/init.d/postgresql reload`
10. Enter virtual environment to load table and initial data for the database:
	Type `cd /var/www/CatalogApp/CatalogApp`
	Type `source venv/bin/activate`
	Type `sudo python database_setup.py`
	Type `sudo python initial_data.py`

## Update Project Code To Deploy

1. Remove Vagrantfile and categories_books_users.db files
2. Update the following variables:
	From `CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']`
	To `CLIENT_ID = json.loads(open('/var/www/CatalogApp/CatalogApp/client_secrets.json', 'r').read())['web']['client_id']`
	From `app_id = json.loads(open('fb_client_secrets.json', 'r').read())['web']['app_id']`
	To `app_id = json.loads(open('/var/www/CatalogApp/CatalogApp/fb_client_secrets.json', 'r').read())['web']['app_id']`
	From `app_secret = json.loads(open('fb_client_secrets.json', 'r').read())['web']['app_secret']`
	To `app_secret = json.loads(open('/var/www/CatalogApp/CatalogApp/fb_client_secrets.json', 'r').read())['web']['app_secret']`
	From ` oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')`
	To ` oauth_flow = flow_from_clientsecrets('/var/www/CatalogApp/CatalogApp/client_secrets.json', scope='')`
3. Change 2 print statements:
	From `print ('No User ID found.')`
	To ```
		nouseridfile = open("/var/www/CatalogApp/CatalogApp/nouseridfile.txt", "a")
        nouseridfile.write('No User ID found')
        nouseridfile.close()```
        **NOTE:** Had to give file permissions (see [here](https://stackoverflow.com/questions/29331872/ioerror-errno-13-permission-denied))
    From `print 'Access Token is None'`
    To ```
    	notokenfile = open("/var/www/CatalogApp/CatalogApp/notokenfile.txt", "a")
        notokenfile.write('Access Token is None')
        notokenfile.close()```
        **NOTE:** Had to give file permissions (see [here](https://stackoverflow.com/questions/29331872/ioerror-errno-13-permission-denied))
 4. Add the following parameters to the book and category tables foreign keys:
 	`onupdate=”CASCADE”, ondelete=”CASCADE”`
 5. Changed connection to database from sql to postgres in database_setup.py, initial_data.py and __init__.py:
 	From `sqlite:///categories_books_users.db`
 	To `postgres://catalog:<password here>@localhost/catalog`
 6. Download and update Google API JSON file (client_secrets.json)
 7. Update Facebook API site URL to `http://34.201.54.34.xip.io`
 	**NOTE:** Facebook now enforce HTTPS (see here)
