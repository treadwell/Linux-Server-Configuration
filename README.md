# Project: Linux Server Configuration

This project describes the steps to set up and configure a remote AWS Lightsail server to host web applications.

## Initial configuration

1. Start a new Ubuntu Linux server instance on Amazon Lightsail, `https://aws.amazon.com/lightsail/`: 
* Sign up
* Select specs
* Download RSA key files

Server IP address is: 54.208.34.181

2. Log into server:

```
ssh -i ~/.ssh/rsa-key.pem ubuntu@AWS_IP_ADDRESS
```

## Secure the server
3. Update all currently installed packages.

```
sudo apt-get update
sudo apt-get upgrade
```

4. Change the SSH port from 22 to 2200: 

* Configure the Lightsail firewall to allow SSH connections on port 2200 on the Network tab of the instance management page.
* Edit `/etc/ssh/sshd_config` to change the SSH port
* Restart the server: `sudo service ssh restart` NB: Do not logout as root to make sure it works first!
* In a separate terminal, check configuration by logging in on the new port: 
```
ssh -i ~/.ssh/rsa-key.pem ubuntu@AWS_IP_ADDRESS -p 2200
```

5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

```
sudo ufw status
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow ntp
sudo ufw allow www
sudo ufw enable
sudo ufw status
```

## Give `grader` access.

6. Create a new user account named grader.

```
sudo adduser grader
```

7. Give grader the permission to sudo.

```
sudo ls /etc/sudoers.d
sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
```
Edit file `grader`, replacing 'ubuntu' with 'grader'.


8. Create an SSH key pair for grader using the ssh-keygen tool.

* On the local machine use ssh-keygen tool create a key for the user `grader`:

```
cd ~/.ssh
ssh-keygen
```
This creates two files. Copy the contents of the .pub file.

* On the remote server (as user `grader`):

```
mkdir ~/.ssh
touch .ssh/authorized_keys

```
Paste the contents of the .pub file from above into the `authorized_keys` file just created.

* Change the permissions on the `.ssh` directory and `authorized_keys` file:

```
chmod 700 .ssh
chmod 664 .ssh/authorized_keys
```

A restart of the ssh service may be required:

```sudo service ssh restart```

* Test the login from the local machine:

```ssh -i ~/.ssh/LightsailCatalog grader@54.208.34.181 -p 2200```

* Disable password-based login, forcing login via the key pair:

Verify entry in the `/etc/ssh/sshd_config` file for PasswordAuthentication is 'no'.

Restart the ssh service: 

```sudo service ssh restart```

## Prepare to deploy your project
9. Configure the local timezone to UTC.

```
sudo dpkg-reconfigure tzdata
# select `None of the above` then `UTC`
```

10. Install and configure Apache to serve a Python mod_wsgi application.

```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi

```

11. Install and configure PostgreSQL:

```
sudo apt-get install postgresql

```

* Create a new database user named `catalog` with limited permissions to the database
* Connect to database as the user postgres: `sudo su - postgres`
* Type `psql` to enter postgres shell
* Create a new user 
`CREATE USER catalog WITH PASSWORD 'password';`
* Confirm that the user was created: `\du` and to verify current permissions.

* Limit permissions to new database user, `catalog`
```
ALTER ROLE catalog WITH LOGIN;
ALTER USER catalog CREATEDB;
```
* Create the database 
```
CREATE DATABASE catalog WITH OWNER catalog;
```
* Login to the database 
```
\c catalog
```
* Revoke all public rights and grant access to only the catalog user:
```
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
```
* Exit the postgres shell and the postgres user 
```
\q
exit
```
* Restart postgres
```
sudo service postgresql restart
```

* Verify that remote connections are not allowed:

```
sudo cat /etc/postgresql/10/main/pg_hba.conf
```

12. Install git

```
sudo apt-get install git
```

## Deploy the Item Catalog project.


13. Clone and setup the Item Catalog project from the Github repository created earlier in the Nanodegree program.

* Create and change permissions on project folders:

```
cd /var/www
sudo mkdir categories
sudo chown grader categories
```

* Clone repository:

```
git clone https://github.com/treadwell/categories.git
```

* Install application dependencies:

```
sudo apt install python-pip
sudo apt-get install python-flask
sudo apt-get install python-sqlalchemy
sudo apt-get install python-psycopg2
sudo apt-get install python-oauth2client
sudo apt-get install python-requests

```


14. Set up the application on the server so that it functions correctly when visiting the server’s IP address in a browser. 

* Validate apache2 install by accessing `http://http://54.208.34.181/` via web browser.  The Apache2 Ubuntu Default Page should be displayed.
* Configure Demo WSGI app by editing the `/etc/apache2/sites-enabled/000-default.conf` file and adding `WSGIScriptAlias / /var/www/html/myapp.wsgi` just before the closing `</VirtualHost>` tag.
* Create the file `/var/www/html/myapp.wsgi` containing:
```
def application(environ, start_response):     
    status = '200 OK'     
    output = 'Hello World!'      
    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]     
    start_response(status, response_headers)      
    return [output]
```
* Restart the apache2 service: `sudo service apache2 restart`

* Validate the wsgi install by accessing `http://http://54.208.34.181/` via web browser.  "Hello World!" should be displayed.

* Configure a WSGI entry point for the catalog app by editing the `/etc/apache2/sites-enabled/000-default.conf` file and changing `WSGIScriptAlias / /var/www/html/myapp.wsgi` to `WSGIScriptAlias / /var/www/categories/myCatalogApp.wsgi` just before the closing `</VirtualHost>` tag.

* Create the file `/var/www/categories/myCatalogApp.wsgi` containing:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/categories")

from application import app as application
application.secret_key = 'super_secret_key'
```

Note that the `app` referred to above is the name of the flask app in `application.py`

* Restart the apache2 service: `sudo service apache2 restart`

* Copy `client_secrets.json` to `/var/www/categories/`

* In `application.py` change path to `client_secrets.json` to  `/var/www/categories/client_secrets.json` in two places

* Update database engine string in `database_setup.py` and `application.py` to `postgresql://catalog@localhost/catalog`

* Update OAUTH for target domain:

	* Go to `https://console.developers.google.com`
	* Make sure that the Catalog app is selected
	* Click on Credentials in the left-hand menu
	* Add Authorized Domain: `http://54.208.34.181.xip.io`
	* Update Authorized Javascript origins to `http://54.208.34.181.xip.io`
	* Update Authorized redirect URIs to `http://54.208.34.181.xip.io/login` and `http://54.208.34.181.xip.io/gconnect`

* Download updated JSON file & rename the file to 'client_secrets.json'

* add JSON file to your project path example: `/var/www/catalog/<your project folder>`


https://knowledge.udacity.com/questions/21110


* Make sure that your .git directory is not publicly accessible via a browser!