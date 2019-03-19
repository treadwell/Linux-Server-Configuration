# Project: Linux Server Configuration

This project describes the steps to set up and configure a remote to host web applications.

## Initial configuration

1. Start a new Ubuntu Linux server instance on Amazon Lightsail. Server IP address is: 54.208.34.181

2. Log into server:

`ssh -i ~/.ssh/rsa-key.pem ubuntu@AWS_IP_ADDRESS`

## Secure the server
3. Update all currently installed packages.

`sudo apt-get update`
`sudo apt-get upgrade`


4. Change the SSH port from 22 to 2200: 

* Configure the Lightsail firewall to allow SSH connections on port 2200 on the Network tab of the instance management page.
* Edit `/etc/ssh/sshd_config` to change the SSH port
* Restart the server: `sudo service ssh restart` ** Do not logout as root to make sure it works first** 
* In a separate terminal, check configuration by logging in on the new port: `ssh -i ~/.ssh/rsa-key.pem ubuntu@AWS_IP_ADDRESS -p 2200`


5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).


## Give `grader` access.

6. Create a new user account named grader.
7. Give grader the permission to sudo.
8. Create an SSH key pair for grader using the ssh-keygen tool.

## Prepare to deploy your project
9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.

* If you built your project with Python 3, you will need to install the Python 3 mod_wsgi package on your server: sudo apt-get install libapache2-mod-wsgi-py3.
11. Install and configure PostgreSQL:

* Do not allow remote connections
* Create a new database user named catalog that has limited permissions to your catalog application database.
12. Install git.

## Deploy the Item Catalog project.
13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!