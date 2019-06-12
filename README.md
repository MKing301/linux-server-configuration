# linux-server-configuration

This project outlines the installation of a Linux server and prepare it to host a web applications, install and configure a database server, and deployment.

*  IP Address: <IP Address>
*  SSH Port: 2200
*  URL:  <URL>

## Resources

* [Amazon Lightsail](https://aws.amazon.com/lightsail/)
* [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Flask](http://flask.pocoo.org/docs/1.0/)
* [SQLAlchemy - PostgreSQL](http://docs.sqlalchemy.org/en/latest/dialects/postgresql.html)
* [Google Oauth 2 Authentication](https://developers.google.com/identity/protocols/OAuth2)

## Setup new Ubuntu Linux server instance on Amazon Lightsail
1.  Go to [Ligthsail](https://aws.amazon.com/lightsail/)
2.  Click Create an AWS Account (yellow button, upper right-hand corner)
3.  Choose an instance image: Ubuntu
4.  Choose your instance plan
5.  Give your instance a hostname
6.  Wait for it to start up
7.  It's running on IPls `t` with the name `linux-catalog`.
8.  Obtain DNS name using the [xip.io](http://xip.io/) service (for OAuth set up). I will use `<URL>`. ([set up DNS Zone in Lightsail](https://lightsail.aws.amazon.com/ls/docs/en/articles/lightsail-how-to-create-dns-entry))
9. Connect using SSH button and update all currently installed packages.
	1. Type `sudo apt-get update`
	2. Type `sudo apt-get upgrade`

## Create a New User

1. Type `sudo adduser grader`
2. Confirm new user using Finger
	1. Type `sudo apt-get install finger`
	2. Type `finger grader`
3. Give `grader` sudo.
	1. Type `sudo nano /etc/sudoers.d/grader`
	2. Enter `grader ALL=(ALL) NOPASSWD:ALL` and save file.
 
## Secure the server

### Local Machine

1. On local machine, generate key pairs:
	* Type `ssh-keygen`
2. Enter file in which to save the key.
3. Enter passphrase.
4. Private and Public key will be saved on local machine in separte files (NOTE: public file has the same name as the private key, but with .pub on the end)

### Remote Server

1. Log into server and change to `grader`
	* Type `su - grader` and enter password at prompt to switch to `grader`
2. In the home directory, add `.ssh` directory
	* Type `mkdir .ssh`
3. Create file to store public key
	1. Type `nano .ssh/authorized_keys`
	2. Copy public key from local machine and paste into the file and save.
4. Change permission for .ssh directory and ssh key file:
	1. Type `chmod 700 ~/.ssh`
	2. Type `chmod 644 ~/.ssh/authorized_keys`
5. Disable password based login
	1. Type `sudo nano /etc/ssh/sshd_config`
	2. Find 'Port 22' and change 22 to 2200
	3. Find 'PasswordAuthentication yes' in the file and 	change 'yes' to �no�
	4. Save and close file.
6. Type `sudo ufw default deny incoming`
7. TYpe `sudo ufw default allow outgoing`
8. Allow ssh by typing `sudo ufw allow ssh`
9. Allow access on port 2200 by typing `sudo ufw allow 2200/tcp`
10. Allow HTTP by typing `sudo ufw allow http`
11. Allow access on port 123 by typing `sudo ufw allow 123/udp`
12. Enable firewall by typing `sudo ufw enable`
13. Restart ssh service
	* Type `sudo service ssh restart`
14. In Lightsail (on you server), go to the Networkingtab to update Firewall settings:
	1. Add Custom TCP 2200
	2. Add Custom UDP 123
	3. Save updates.
15. From local machine, verify you can SSH into server using port 2200:
	1. Type ` ssh grader@<IP Address> -p 2200 -i ~/.ssh/linuxProject`
	2. Enter passphrase for key
	3. Successfully logged in as `grader` via SSH
16.	In Lightsail (on you server), go to the Networking tab to edit Firewall settings:
	* Remove SSH TCP 22

## Prepare to deploy the project

### Set timezone to UTC

1.  Configure the local timezone to UTC.
	1. Type `sudo dpkg-reconfigure tzdata`
	2. Scroll to the bottom of the Continents list and select None of the above
	3. In the second list, select UTC
	source: [https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt](https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt)
	
### Install Apache

1. Install apache
	* Type `sudo apt-get install apache2`
2. Install and Enable mod_wsgi
	1. Type `sudo apt-get install libapache2-mod-wsgi`
	2. Type `sudo a2enmod wsgi`
	
### Install Git

1. Install git
	* Type `sudo apt-get install git`
	
### Set up application structure

1. Create app
	1.  Type `cd /var/www`
	2.  Type `sudo mkdir CatalogApp`
	3.  Type `cd CatalogApp`
	4.  Clone repository from Catalog Project (completed earlier in course)
		*  Type `sudo git clone https://github.com/MKing301/catalog.git CatalogApp`
	5.  Rename `catalog_app.py` to `__init__.py`
		1.  Type `cd CatalogApp`
		2.  Type `sudo mv catalog_app.py __init__.py`
2. Make sure that your .git directory is not publicly accessible via a browser (see [here](https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible))
	 
### Install Flask and Set-Up Virtual Environment

1. Install PIP
	* Type `sudo apt-get install python-pip`
2. Install virtualenv
	* Type `sudo pip install virtualenv`
3. Name virtual environment
	* Type `sudo virtualenv venv`
4. Activate vitual environment
	* Type `source venv/bin/activate` NOTE: (venv) will appear at begining of line
5. Install Flask here
	* Type `sudo pip install Flask`
6. Type `deactivate` to exit virtual environment

### Install additional required packages

1. Install all of the following:
	1. Type `sudo apt-get install python-psycopg2`
	2. Type `sudo pip install psycopg2-binary`
	3. Type `sudo pip install SQLAlchemy`
	4. Type `sudo pip install Flask-SQLAlchemy`
	5. Type `sudo pip install requests`
	6. Type `sudo pip install httplib2`
	7. Type `sudo pip install flask_httpauth`
	8. Type `sudo pip install oauth2client`

### Configure and Enable Virtual Host

1. Type `sudo nano /etc/apache2/sites-available/CatalogApp.conf`
2. Edit file to contain the following:
	```
	<VirtualHost *:80>
		ServerName <your server name>
		ServerAdmin admin@<your server name>.com
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
	1. Type `cd /var/www/CatalogApp`
	2. Type `sudo nano catalogapp.wsgi`
	3. Add the following lines in the file:
		```
		#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0,"/var/www/CatalogApp/")
		from CatalogApp import app as application
		application.secret_key = 'super secret key'
		```
		**NOTE:** Obtain secret key (check [here](https://gist.github.com/geoffalday/2021517))
5. Disable default:
	* Type `sudo a2dissite 000-default.conf`
6. Add ip and host
	1. Type `sudo nano /etc/hosts`
	2. Add `<IP Address> <your server name>`
7. Enable virtual host:
	* Type `sudo a2ensite CatalogApp`
8. Restart apache
	* Type `sudo service apache2 restart`
	
## Install and Set-Up Database

1. Type `sudo apt-get install postgresql` to install
2. Type `sudo -u postgres psql postgres` to access postgreSQL
3. Type `\password postgres` to set the password
4. Type `CREATE DATABASE catalog;` to create the database
5. Type `\c catalog` to connect to the database
6. Create user catalog by typing `Create a user with: postgres=# CREATE USER <username> WITH PASSWORD '<password>';`
7. Grant user catalog permission to database catalog:
	* Type `GRANT ALL PRIVILEGES ON DATABASE <database name> TO <username>;`
8. Type `\q` then `ENTER` to exit postgreSQL
9. Reload postgresql by type `sudo /etc/init.d/postgresql reload`
10. Enter virtual environment to load table and initial data for the database:
    1. Type `cd /var/www/CatalogApp/CatalogApp`
	2. Type `source venv/bin/activate`
	3. Type `sudo python database_setup.py`
	4. Type `sudo python initial_data.py`

## Update Project Code To Deploy

1. Remove Vagrantfile and categories_books_users.db files
2. Update the following variables:
    1. From `CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']`
    2. To `CLIENT_ID = json.loads(open('/var/www/CatalogApp/CatalogApp/client_secrets.json', 'r').read())['web']['client_id']`
    3. From `app_id = json.loads(open('fb_client_secrets.json', 'r').read())['web']['app_id']`
    4. To `app_id = json.loads(open('/var/www/CatalogApp/CatalogApp/fb_client_secrets.json', 'r').read())['web']['app_id']`
    5. From `app_secret = json.loads(open('fb_client_secrets.json', 'r').read())['web']['app_secret']`
    6. To `app_secret = json.loads(open('/var/www/CatalogApp/CatalogApp/fb_client_secrets.json', 'r').read())['web']['app_secret']`
    7. From ` oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')`
    8. To ` oauth_flow = flow_from_clientsecrets('/var/www/CatalogApp/CatalogApp/client_secrets.json', scope='')`
3. Change 2 print statements:
    1. From `print ('No User ID found.')`
    2. To```
		nouseridfile = open("/var/www/CatalogApp/CatalogApp/nouseridfile.txt", "a")
        nouseridfile.write('No User ID found')
        nouseridfile.close()```
        **NOTE:** Had to give file permissions (see [here](https://stackoverflow.com/questions/29331872/ioerror-errno-13-permission-denied))
    3. From `print 'Access Token is None'`
    4. To ```
    	notokenfile = open("/var/www/CatalogApp/CatalogApp/notokenfile.txt", "a")
        notokenfile.write('Access Token is None')
        notokenfile.close()```
        **NOTE:** Had to give file permissions (see [here](https://stackoverflow.com/questions/29331872/ioerror-errno-13-permission-denied))
 4. Add the following parameters to the book and category tables foreign keys:
    * `onupdate='CASCADE', ondelete='CASCADE'`
 5. Changed connection to database from sql to postgres in database_setup.py, initial_data.py and __init__.py:
    1. From `sqlite:///categories_books_users.db`
    2. To `postgres://catalog:<password here>@localhost/catalog`
 6. Download and update Google API JSON file (client_secrets.json)
 7. Removed Facebook Oauth Authentication
	**NOTE:**  Facebook now enforce HTTPS (see [here](https://developers.facebook.com/blog/post/2018/06/08/enforce-https-facebook-login/))
