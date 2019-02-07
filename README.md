# udacity-linux-server-configuration

## Getting started:
the grader user information (private key / password) are submitted with the project 

- catalog is deployed under: [http://catalog.udacity.swmo.ch/]: http://catalog.udacity.swmo.ch/
- neighborhood-map is deployed under: [http://neighborhood-map.udacity.swmo.ch/]:  

## Documentation:

### ssh access:
first we create a ssh key to connect the aws instance.
it will genarate a private key and and public key
```
ssh-keygen -t rsa -b 4096
```

now open aws terminal. paste the public key (*.pub) key to the authorized_keys and save the file.

```
nano .ssh/authorized_keys
```

now we are able to login from external with our private key;
```
ssh ubuntu@52.29.225.13 -i ~/.ssh/<private key file>
```

### change ssh port:
if we change the ssh port we sould first open the new port on  the aws firewall:

![alt text](resources/screenshots/aws_firewall.png "AWS Firewall")


now we are chaging the port from 22 to 2200 in the ssh config on the server.
open the ssh config file:
```
sudo nano /etc/ssh/sshd_config
```

change the listen port to 2200:
![alt text](resources/screenshots/sshd_config.png "sshd config")

make also sure that only login with key pair is allowed:
so:

```
PasswordAuthentication no
```

and also no ssh root login is allowed:

```
PermitRootLogin no #prohibit-password would mean that it's possible over a key pair
```

restart the ssh service so the port changes happens:
```
sudo /etc/init.d/ssh restart
```

now test from the client if you can use the port 2200 for ssh connecting:
```
ssh ubuntu@52.29.225.13 -p 2200 -i .ssh/<private key file>
``` 

### firewall
we should only allow followin connections:
- SSH (port 2200) -> tcp
- HTTP (port 80) -> tcp
- NTP (port 123) -> udp

we have two firewalls: 
- the aws firwall 
- the server firewall "ufw"

for this course i will configure both firwalls similar:

1. AWS Firewall
![alt text](resources/screenshots/aws_firewall_config.png "AWS Firewall")

2. ufw Firewall
make sure the firewall is not enabled: (otherwise you could block yourself out)
```
sudo ufw status
```

Now start with configuration, first block all incoming:
```
sudo ufw default deny incoming
```

Allow all outgoing:
```
sudo ufw default allow outgoing
```

Allow ingcoming ssh:
```
sudo ufw allow 2200/tcp
```
Allow incoming http:
```
sudo ufw allow 80/tcp
```
Allow incoming ntp:
```
sudo ufw allow 123/udp
```
Now we can activate the ufw firewall:
```
sudo ufw enable
```

Now we can check the configuration:
```
sudo ufw status
```
here we see our defined rules:
![alt text](resources/screenshots/ufw_status.png "UFW Firewall")

### Grader User
now lets create the grader user and add him to the sudo group

```
sudo adduser grader
```

Add the user to the sudo group:

```
sudo usermod -aG sudo grader
```

Now generation a ssh key for the grader user: (on the laptop)

```
ssh-keygen -t rsa -b 4096
```

on the server add the public key to the authorized_keys file of the grader user:

```
su grader
mkdir ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

now your able to login over ssh:
```
ssh grader@52.29.225.13 -i .ssh/<private key file> -p 2200
```

### Updates
now let's update the hole system:
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
```

### Install apache

```
sudo apt-get install apache2
```

Now the apache already returns a default website: 
http://52.29.225.13/


### Install python apache modul

for the catalog app i used python 2.7, so we install the apache modul (if we used python3 the package would be: libapache2-mod-wsgi-py3).
It will install all dependencies like (libapache2-mod-wsgi libpython-stdlib libpython2.7 libpython2.7-minimal libpython2.7-stdlib python python-minimal python2.7 python2.7-minimal)

```
sudo apt-get install libapache2-mod-wsgi
```

### Setup Postgresql

for the catalog we will also need a postgresql.
```
sudo apt-get install postgresql
```

create catalg database and user:
```

sudo -u postgres createuser catalog
sudo -u postgres createdb catalog
sudo -u postgres psql
alter user catalog with encrypted password '***';
grant all privileges on database catalog to catalog;

```

### Install python

we also need some python packages:

Install python packages:

```
sudo apt-get install python-pip
sudo pip install flask
sudo pip install flask_wtf
sudo pip install flask_bcrypt
sudo pip install python-resize-image
sudo pip install oauth2client
sudo pip install sqlalchemy
```

### deploy catalog
we will deploy the site under /var/www/catalog.udacity.swmo.ch/
here we use my domain swmo.ch, so i had to create an DNS A Record which points to the aws server (change to static IP in aws)

prepare the directories:
```
cd /var/www/
mkdir catalog.udacity.swmo.ch
```

now deploy the code. i had to do a few changes (postgres instead of sqllite), and a public folder (for document root) better than use the hole python code under the document root.
thats the reason we i use in this execirse an "deploy" branch.
so lets checkout the code:

```
sudo git clone https://github.com/swmo/udacity-catalog.git .
sudo git checkout deploy
```

we need to have an a wsgi file, which we will use with apache:

/var/www/catalog.udacity.swmo.ch/catalog.wsgi

```
#!/usr/bin/python
import sys
sys.path.insert(0, '/var/www/catalog.udacity.swmo.ch/')

from app import app as application
application.root_path = '/var/www/catalog.udacity.swmo.ch/'
application.secret_key = 'fklsjfdlajiejrkaejrklajlnIE((*HFUFHU'
```

now we can start to configure apache for the catalog site

```
cd /etc/apache2/sites-available
sudo cp 000-default.conf catalog.udacity.swmo.ch.conf
```

catalog.udacity.swmo.ch.conf
- ServerName: need to be the domain / subdomain
- DocumentRoot: points to the public folder, so the other python code is better protected
- set the Error and Access Log path.
- define under which user the wsgi run
- let apchae acces the public folder (require all granted)
```
<VirtualHost *:80>
	ServerName catalog.udacity.swmo.ch

	ServerAdmin moses.tschanz@gmail.com
	DocumentRoot /var/www/catalog.udacity.swmo.ch/public

	ErrorLog ${APACHE_LOG_DIR}/catalog.udacity.swmo.ch.error.log
	CustomLog ${APACHE_LOG_DIR}/catalog.udacity.swmo.ch.access.log combined

	WSGIDaemonProcess catalog user=www-data group=www-data threads=5

	Alias /static/ /var/www/catalog.udacity.swmo.ch/public/static/
	<Directory /var/www/catalog.udacity.swmo.ch/public/static>
		Require all granted
	</Directory>
	WSGIScriptAlias / /var/www/catalog.udacity.swmo.ch/public/catalog.wsgi
	<Directory /var/www/catalog.udacity.swmo.ch/public>
		Require all granted
	</Directory>

</VirtualHost>
``

Now we can activate the site
```
sudo a2ensite catalog.udacity.swmo.ch
sudo service apache2 reload
```

###deploy neighborhood-map:

to deploy the neighborhood-map is easier, because its just html / js and css.

first we deploy the code:

```
cd /var/www/
sudo mkdir neighborhood-map.udacity.swmo.ch
cd neighborhood-map.udacity.swmo.ch
#all files need to be public:
sudo git clone https://github.com/swmo/udacity-neighborhood-map.git public
```

now we can make a new site

```
cd /etc/apache2/sites-available
sudo cp 000-default.conf neighborhood-map.udacity.swmo.ch.conf
```

/etc/apache2/sites-available/neighborhood-map.udacity.swmo.ch.conf:
```
<VirtualHost *:80>
        ServerName neighborhood-map.udacity.swmo.ch
        ServerAdmin moses.tschanz@gmail.com
        DocumentRoot /var/www/neighborhood-map.udacity.swmo.ch/public
        ErrorLog ${APACHE_LOG_DIR}/neighborhood-map.udacity.swmo.ch.error.log
        CustomLog ${APACHE_LOG_DIR}/neighborhood-map.udacity.swmo.ch.access.log combined
        <Directory /var/www/neighborhood-map.udacity.swmo.ch/public>
                Require all granted
        </Directory>
</VirtualHost>
```

Now we can activate the site
```
sudo a2ensite neighborhood-map.udacity.swmo.ch
sudo service apache2 reload
```


### Permisson:
We also have to set the direcory 

now change owner of the directory:
allow minimal rights
```
sudo chown -R www-data:www-data /var/www/html
sudo chown -R www-data:www-data /var/www/catalog.udacity.swmo.ch
sudo chmod -R 550 /var/www/catalog.udacity.swmo.ch
#make sure the server cat write to to store the profile images (upload)
sudo chmod -R 770 /var/www/catalog.udacity.swmo.ch/public/static/uploads

sudo chown -R www-data:www-data /var/www/neighborhood-map.udacity.swmo.ch
sudo chmod -R 550 /var/www/neighborhood-map.udacity.swmo.ch
```

### secure Apache
```
sudo nano /etc/apache2/apache2.conf
```
only return minimal information to the client
```
ServerSignature Off
ServerTokens Prod
```

restrict access to all directory:
```
<Directory />
   Options None
   Require all denied
</Directory>
```

Disable List of files in /var/www/html, but grant access for the webserver
```
<Directory /var/www/html>
    Options -Indexes
	Require all granted
</Directory>
```

make sure the apache runs under his own user / group:

```
cat /etc/apache2/envvars

it should be like: not root!!
export APACHE_RUN_USER=www-data
export APACHE_RUN_GROUP=www-data
```

```
sudo service apache2 restart
```

## good to know
- added an google-site-verification to proof the domain is mine



## sources of information:
- udacity course (Deploying to Linux Servers)
- https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
- http://www.linuxlookup.com/howto/change_default_ssh_port
- https://blog.buettner.xyz/sichere-ssh-konfiguration/
- https://www.tecmint.com/apache-security-tips/
- https://www.bogotobogo.com/python/Flask/Python_Flask_HelloWorld_App_with_Apache_WSGI_Ubuntu14.php