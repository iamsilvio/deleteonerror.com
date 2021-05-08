# SCP only configuration

https://blog.tinned-software.net/setup-sftp-only-account-using-openssh-and-ssh-key/
https://www.jujens.eu/posts/en/2018/Sep/09/deploy-test-git-hooks/

## local group

``` shell
groupadd sftponly
```

## local user

Create a user login permissions
useradd -G sftponly -s /sbin/nologin user1


in our case we want the user to be able to deploy files to a webserver directory so we add the user also to www-data

``` shell
usermod -a -G www-data autoDeploy
```


## ssh Daemon config

``` conf



```