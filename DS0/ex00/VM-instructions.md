#

apk add curl

## Entorno Grafico en Alpine
setup-wayland-base labwc foot alacritty firefox mako
rc-update add elogind boot
service elogind start
rc-update add dbus boot
service dbus start
apk add pam-rundir
apk add font-dejavu font-noto font-awesome

## Navegador para labwc
```bash
apk add epiphany
```

### Keyboard shortcuts
I created the file ~/.config/labwc/rc.xml with this configuration

Windows + b for Browser

```xml

<?xml version="1.0"?>
<labwc_config>
  <keyboard>
    <keybind key="W-b">
      <action name="Execute" command="epiphany" />
    </keybind>
  </keyboard>
</labwc_config>
```


# Create user

Inside the virtual machine I defined a user with same UID and GID that the ones I hold in 42Barcelona

+ 1.- Check who am I in the host machine.
```bash  
id luicasad 
uid=101177(luicasad) gid=4223(2023_barcelona) groups=204(_developer),4223(2023_barcelona)
```

+ 2.- Replicate my character in the VM.
```bash
addgroup -g 4223 2023_barcelona
adduser -u 101177 -G 2023_barcelona -D luicasad
passwd luicasad
```

+ 3.- Create ssh key
  
I save in `~/.ssh` a ssh key to conect with github. I named it `alpine_piscine`
```bash
ssh-keygen -t ed25519 -C "luismiguelcasadodiaz@gmail.com"

cat ~/.ssh/alpine_piscine.pub
```

+ 4.- start ssh-agent when login

I created a `~/.profile` file that executes at login time. That ensures the
`ssh-agent` has my `ssh key`
```bash
echo "Logged in as: $(whoami)"
echo "Hostname    : $(hostname)"
echo "Kernel      : $(uname -r)"
echo "Postgresql  : $(psql --version)"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/alpine_piscine
```

+ 5.- copy 42 ssh keys from host machine into VM
I opened a ssh session from host to VM in a terminal i call `remote` to help me wiht this explanation.

Inside a `local` terminal in Host machine I copied `~/.ssh/id_rsa`'s content.
In the `remote` terminal, with Vim, after setting mouse wiht `set mouse=n` y pasted the key.

Repeat for `id_rsa.pub`.

+ 6 allow user to set a Graphic interface 
addgroup <tu_usuario> video
addgroup <tu_usuario> input
addgroup <tu_usuario> seat



# Install git

Install it is simple `apk add git`. 
You must add `~/.ssh/alpine_piscine.pub` to GitHub




# Install Postgresql

To install postgresql in alpine execute this command

```bash
# apk add postgresql18 postgresql18-contrib
```

Installation also creates a new user `postgres` with home folder `\var\lib\postgresql`

Define a service named `postgresql` to start at boot time
```bash
# rc-update add postgresql
# rc-service postgresql start
```

Login as the `postgres` user and start `psql` to create a new user and database according to subject requirements:

```bash
su postgres
psql
create user luicasad with encrypted password 'mysecretpasswd';
create database piscineds;
grant all privileges on database piscineds to luicasad;
```

### Host-based Authentication

The file `pg_hba.conf` defines a table with rules for postgresql about who can connect from where, to which database and using which identiy test.

I changed 
TYPE        DATABASE    USER        ADDRESS         METHOD
host        all         all         127.0.0.1/32    scram-sha-256
host        all         all         ::1/128     scram-sha-256

to 

host        piscineds       luicasad        127.0.0.1/32    scram-sha-256
host        piscineds       luicasad        ::1/128         scram-sha-256

cause:
scram-sha-256 requiers a verified password
host relates to a tcp/ip conneciton either encrypted or no.

### Graphic customer

#### Install PHP with PostgreSQL support
```sh
apk add php83 php83-pdo php83-pdo_pgsql php83-session php83-json php83-openssl
```

#### Download adminer

```sh
mkdir -p ~/adminer && cd ~/adminer
curl -L https://www.adminer.org/latest.php -o adminer.php
```
#### Configure  bootable adminer rc-service

I create the `/etc/init.d/adminer` as `su` with execution permission.
```sh
#!/sbin/openrc-run

name="adminer"
description="Servidor web de Adminer (PHP built-in server)"

command="/usr/bin/php83"
command_args="-S 127.0.0.1:8080 -t /home/luicasad/adminer"
command_user="luicasad:users"
command_background="yes"
pidfile="/run/${RC_SVCNAME}.pid"

depend() {
    need net
}
```

I add it to boot runlevel
```sh
rc-update add adminer boot
``` 

