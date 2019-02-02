# udacity-linux-server-configuration


## setup:

create a ssh key to connect the aws instance:
```ssh-keygen -t rsa -b 4096```

it will genarate a private key and and public key

open aws terminal

```
nano .ssh/authorized_keys
```

paste the public key (*.pub) key to the authorized_keys and save the file.


