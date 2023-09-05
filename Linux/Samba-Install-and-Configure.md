# Samba Install and Configure

## Install samba

``` bash
sudo apt update
sudo apt install samba

# check status
sudo systemctl status smbd
```

## Configure firewall (optional)

``` bash
sudo ufw allow 'Samba'
```

## Create Backup of configuration files

``` bash
sudo cp /etc/samba/smb.conf{,.backup}
```

## Configuration

``` bash
sudo nano /etc/samba/smb.conf
```

Defailt values

* `server role = standalone server`
* `interfaces = 127.0.0.0/8 eth0` Listen on all interfaces
* `bind interfaces only = yes`

Ater changes you can test config file with `testparm`. If everythinq si OK you will see `Loaded services file OK`

Restart Samba services with:

``` bash
sudo systemctl restart smbd
sudo systemctl restart nmbd
```

## Folders

Instead of using default folder create new location for example `/samba`

``` bash
sudo mkdir /samba
```

And change group to default samba's group `sambashare`

``` bash
sudo chgrp sambashare /samba
```

## Create samba users

Create users with following command

``` bash
sudo useradd -M -d /samba/maymeow -s /usr/sbin/nologin -G sambashare maymeow
```

* `-M` do not create home directory
* `-d /samba/maymeow` set users directory to `/samba/maymeow`
* `-s /usr/sbin/nologin` disable shell access to th
* `-G sambashare` add user to `sambashare` group

Create home directory for this user and chage directory permission

``` bash
sudo mkdir /samba/maymeow
sudo chown maymeow:sambashare /samba/maymeow

sudo chmod 2770 /samba/josh
```

Add user to samba by setting password for account

``` bash
sudo smbpasswd -a maymeow
```

Once you have password you can enable account

``` bash
sudo smbpasswd -e maymeow
```

You can create more users by following this seps if you need them.

## Configuring Samba shares

``` bash
sudo nano /etc/samba/smb.conf
```

``` bash
[users]
    path = /samba/users
    browseable = yes
    read only = no
    force create mode = 0660
    force directory mode = 2770
    valid users = @sambashare @sadmin

[josh]
    path = /samba/maymeow
    browseable = no
    read only = no
    force create mode = 0660
    force directory mode = 2770
    valid users = maymeow @sadmin
```

* `path` path to share
* `browseable` whether thw folder is listed or no in shares list. By setting to `no` users cannot see this share in list
* `read only` whether the `valid users` have abilty to write to this folder.
* `force create mode` permissions for newly created files
* `force directory mode` permissions for newly created folders
* `valid users` list of users that are allowed to acces this folder. one with `@` prefix are groups