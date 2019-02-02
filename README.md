# udacity-linux-server-configuration


## setup:

- create a ssh key to connect the aws instance.
it will genarate a private key and and public key
```ssh-keygen -t rsa -b 4096```


- open aws terminal. paste the public key (*.pub) key to the authorized_keys and save the file.

```
nano .ssh/authorized_keys
```



