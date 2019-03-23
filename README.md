# Project: Linux Server Configuration
This project sets up a Linux server (an Amazon Lightsail Ubuntu instance) with good security settings, a running web server, and deployment of the earlier Item Catalog project.

## Summary
IP address: 34.219.63.62

SSH port: 2200

Host Name (Complete URL to the web application): http://ec2-34-219-63-62.us-west-2.compute.amazonaws.com

## Google OAuth Instructions:
- Go to  [Google Devloper Console](https://console.developers.google.com/)
- Add http://ec2-34-219-63-62.us-west-2.compute.amazonaws.com to trusted host
- Create Crednetials, select OAuth client ID then  Web Application 
- Under Authorized JavaScrip Origins enter http://ec2-34-219-63-62.us-west-2.compute.amazonaws.com
- Under Authorized redirect URIs enter http://ec2-34-219-63-62.us-west-2.compute.amazonaws.com and http://ec2-34-219-63-62.us-west-2.compute.amazonaws.com/gconnect
- Download the client_secrets.json file

## Server configuration
#### Setup Lightsail:
- Create an Ubuntu instance at [Amazon Lightsail](https://lightsail.aws.amazon.com)
- Go to Networking tab, under firewall add UDP 123 and TCP 2200

#### Install and update software packages:
$ `sudo apt-get update`

$ `sudo apt-get upgrade`

$ `sudo do-release-upgrade`

$ `sudo apt-get install finger`

$ `sudo apt-get install apache2`

$ `sudo apt-get install postgresql`

$ `sudo apt-get install libapache2-mod-wsgi`

$ `sudo apt autoremove`

$ `sudo apt-get install git`

$ `sudo apt-get install python-pip`

$ `sudo pip install Flask`

$ `sudo pip install --upgrade pip`

$ `sudo pip install --upgrade google-api-python-client`

$ `sudo pip install --upgrade google-auth google-auth-oauthlib google-auth-httplib2`

$ `sudo pip install --upgrade flask`

$ `sudo pip install --upgrade requests`

$ `sudo pip install sqlalchemy`

$ `sudo apt install upstart`

$ `sudo apt-get install postgresql postgresql-contrib`

$ `sudo pip install --upgrade pip`

$ `sudo pip install virtualenv`

$ `sudo apt-get install libpq-dev python-dev`

$ `sudo apt install unattended-upgrades`

$ `sudo dpkg-reconfigure --priority=low unattended-upgrades`

$ `sudo apt-get dist-upgrade`

#### Set up the firewall and security settings:
$ `sudo nano /etc/ssh/sshd_config`
    change port from 22 to 2200 
    
$ `sudo nano /etc/init.d/sshd restart`

$ `sudo ufw allow 2200/tcp`

$ `sudo ufw allow 80/tcp`

$ `sudo ufw allow 123/udp`

$ `sudo ufw default deny incoming`

$ `sudo ufw default allow outgoing`

$ `sudo ufw allow ntp`

$ `sudo ufw added`

$ `sudo ufw enable`

$ `sudo ufw status`

#### Add 'grader' to sudo users:
$ `sudo adduser grader`

$ `sudo nano /etc/sudoers.d/grader`  and add the following line to this file:

grader ALL=(ALL) NOPASSWD:ALL
Generate keypair on local machine using `ssh-keygen`, and copy the content of `grader.pub` file

$ `sudo su grader`

$ `mkdir .ssh`

$ `nano .ssh/authorized_keys` and paste the content of the `grader.pub` file

$ `chmod 700 .ssh`

$ `chmod 644 .ssh/authorized_keys`

Now the server can be accessed by: `ssh grader@34.219.63.62 -i [graderPrivateKey.pem] -p 2200`

## Virtual host setup and configuration
#### Setup the PostgreSql database and create a new database user 'catalog':
$ `sudo -u postgres createuser -P catalog`

$ `sudo -u postgres createdb -O catalog catalog`

$ `sudo -u postgres -i`

- Git clone the catalogApp project in to a catalog directory
- Create and fill the database:

  $ `sudo nano database_setup.py`

  $ `sudo nano categories_setup.py`

#### Install virtual environment:
$ `sudo virtualenv venv`

$ `source venv/bin/activate`
    
$ `python3 -m venv env`
    
$ `sudo chmod -R 777 venv`
    
$ `sudo pip install Flask`
    
$ `sudo pip install bleach httplib2 request oauth2client sqlalchemy`
    
$ `pip install psycopg2`
    
$ `sudo python database_setup.py`
    
$ `sudo python categories_setup.py`
    
$ `sudo a2ensite catalog`

#### Enable the virtual host to host the .wsgi application:
- Create catalog.wsgi file:
$ `sudo nano /var/www/catalog/catalog.wsgi`, and add the following [1]:
    #!/usr/bin/python
    activate_this = '/var/www/catalog/catalogApp/venv/bin/activate_this.py'
    execfile(activate_this, dict(\__file__=activate_this))
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'your_secret_key'
- Place the client_secrets.json file properly inside the project (the \__init__.py and templates/login.html need to be updated accordingly[2])
- Enable virtual host:
$ `sudo nano /etc/apache2/sites-available/catalog.conf` and add the following[3]:

    <VirtualHost *:80>
         ServerName 34.219.63.62
         ServerAdmin admin@34.219.63.62
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
- $ `sudo service apache2 restart`
## References
[1] https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

[2] https://developers.google.com/identity/sign-in/web/server-side-flow

[3] https://github.com/bad2thuhbone/Linux-Server-Project-6

## Project Rubric

|SECTION||CRITERIA|SPECS. MET?|
|---|---|---|---|
| User Management | Can you log into the server as the user `grader` using the submitted key? | The SSH key submitted with the project can be used to log in as grader on the server. |Yes|
| | Is remote login of the `root` user disabled? | You cannot log in as root remotely. |Yes|
| | Is the grader user given `sudo` access? | The `grader` user can run commands using `sudo` to inspect files that are readable only by root.|Yes|
| Security | Is the firewall configured to only allow for `SSH`, `HTTP`, and` NTP`? | Only allow connections for `SSH` (port 2200), `HTTP` (port 80), and `NTP` (port 123). |Yes|
| | Are users required to authenticate using `RSA` keys? |Key-based `SSH` authentication is enforced.|Yes|
| | Are the applications up-to-date? | All system packages have been updated to most recent versions. |Yes|
| | Is `SSH` hosted on non-default port? | SSH is hosted on non-default port. |Yes|
| Application Functionality | Is there a web server running on port 80? |The web server responds on port 80.|Yes|
| | Has the database server been configured to properly serve data? | Database server has been configured to serve data (PostgreSQL is recommended).|Yes|
| | Has the web server been configured to serve the Item Catalog application? | Web server has been configured to serve the Item Catalog application as a WSGI app.|Yes|
| Documentation | Is a README file included in the GitHub repo containing all specified information? |A README file is included in the GitHub repo containing the following information: IP address, URL, summary of software installed, summary of configurations made, and a list of third-party resources used to complete this project.|Yes|
