# udacity-linux-server-configuration


## Setup:

### ssh access:
first we create a ssh key to connect the aws instance.
it will genarate a private key and and public key
```ssh-keygen -t rsa -b 4096```

now open aws terminal. paste the public key (*.pub) key to the authorized_keys and save the file.

```
nano .ssh/authorized_keys
```

now we are able to login from external with our private key;
```
ssh ubuntu@52.29.225.13 -i ~/.ssh/<private key file>
```

### change ssh port:
if we change the ssh port we sould first open it at the aws firewall:

![alt text](resources/screenshots/aws_firewall.png "AWS Firewall")


now we are chaging the port from 22 to 2200
open the ssh config file:
```
nano /etc/ssh/sshd_config
```

change the listen port to 2200:
```






```
adduser grader

``



## sources of information:
- udacity course (Deploying to Linux Servers)
- https://www.digitalocean.com/community/tutorials/how-to-create-a-sudo-user-on-ubuntu-quickstart
- http://www.linuxlookup.com/howto/change_default_ssh_port