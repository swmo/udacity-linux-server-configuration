# udacity-linux-server-configuration


## Setup:

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
``

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
```¨

### Install apache

```
sudo apt-get install apache2
```

Now the apache already returns a default website: 
http://52.29.225.13/

#### secure Apache
```
sudo nano /etc/apache2/apache2.conf
```
only return minimal information to the client
```
ServerSignature Off
ServerTokens Prod
```


Disable List of files
```
<Directory /var/www/html>
    Options -Indexes

    <FilesMatch "^index\.">
	    Order allow,deny
	    allow from all
   </FilesMatch>
</Directory>
```

restrict access to directory:
make sure you have added the filesMatch in <Directory /var/www/html> .. </Direcotry> otherwise your index file is not accessible anymore.
```
<Directory />
   Options None
   Order deny,allow
   Deny from all
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
``






## sources of information:
- udacity course (Deploying to Linux Servers)
- https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
- http://www.linuxlookup.com/howto/change_default_ssh_port
- https://blog.buettner.xyz/sichere-ssh-konfiguration/
- https://www.tecmint.com/apache-security-tips/