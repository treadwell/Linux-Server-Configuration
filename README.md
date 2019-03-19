# Project: Linux Server Configuration

This project describes the steps to set up and configure a remote to host web applications.

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

* Change to postgres user:

```sudo -i -u postgres```

* Create new dasabase user `catalog`:

```
createuser --interactive -P

Enter name of role to add: catalog
Enter password for new role:
Enter it again:
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) n
Shall the new role be allowed to create more new roles? (y/n) n
```

* Create new database `catalog`:

```
psql
CREATE DATABASE catalog;
\q
```

* logout postgres user:


```
exit
```

12. Install git

```
sudo apt-get install git
```

## Deploy the Item Catalog project.


13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.

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
pip install psycopg2
pip install flask
pip install sqlalchemy
pip install oauth2client
pip install requests
```


14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!