## Version

# Virtual Box

I created the virtual machine with With Oracle Virtual BOx Manager version 7.0.

Name: AlpinePiscine
Location : /media/lcasado/iDiskk/VM (a 250 GB gift won at 2026 Talent Arena)
ISO image:/home/lcasado/Downloads/alpine-virt-3.24.0-x86_64.iso. It is a 64-bit linux 2.6 distribution.
4MB of Base Memory.
3 CPUs

64MB for video memory
VMSVGA Graphics controller.

Pre-allocate the full size of a 20 GB Hard Disk
Enable a network adapter attached to a Bridged Adpater. This will allow the virtual machine to behave as a brother machine of the host machine

# Image

I use an Alpine Linux version with a slimmed-down kernel. Optimized for virtual systems.[Named virtual ](https://dl-cdn.alpinelinux.org/alpine/v3.24/releases/x86_64/alpine-virt-3.24.0-x86_64.iso) releases in jun 2026 with 69 MB

It is a volatile version designed to exist only in RAM. Running setup, you MUST make the file system persistent

# Login
User root has no password. FIX this at setup.

# setup-alpine
This is the script to configure Alpine Linux
+ select keyboard map: us us ---> to fit with 42 Mac keyboards
+ select hostname: localhost [default]
+ Interface:
    + to initialize: eth0 [default]
    + IPv4 address for eth0: dhcp [default]
    + IPv6 address for eth0: auto [default]    
    + any manual network configuration: n [default]

+ Root Password:
+ TimeZone: Europe Madrid
+ Proxy: none [default]
+ Network Time Protocol: busybox [default]
+ APK Mirror
    + Enable community repositories (c). Required to download Docker, Git, etc...
    + Find and use the fastest mirror (f) --> mirror.raiolanetworks.net Lugo, Spain  NUMBER 84
+ user: no  [default] --> I will create it later with the UID/GID luicasad has in hostmachine
+ ssh
    + server: openssh [default]
    + root login prohibit-password [default]
    + root key: nono [default]
+ Disk & Install
    + disk to use: sda
    + how to use it: sys  --> I want SDA to become a system disk.
    + Erase sda disk and continue: y

# Create user, ssh key and .profile

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
+ 3.- Create ssh key inside the virtual machine

login as luicasad
  
I save in `~/.ssh` a ssh key to conect with github. I named it `alpine_piscine`
```bash
ssh-keygen -t ed25519 -C "luismiguelcasadodiaz@gmail.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/luicasad/.ssh/id_ed25519): 
Created directory '/home/luicasad/.ssh'.
Enter passphrase for "/home/luicasad/.ssh/id_ed25519" (empty for no passphrase):
Created directory '/home/luicasad/.ssh'.
Enter passphrase for "/home/luicasad/.ssh/id_ed25519" (empty for no passphrase): 

```
+ 4 Copy public key to github

```
 cat ~/.ssh/id_ed25519.pub 
```

+ 5.- start ssh-agent when login

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

# Install packages as root
+ 1.- git

apk add git
apk add curl
apk add make
apk add python3
apk add postgresql18 postgresql18-contrib
apk add php83 php83-pdo php83-pdo_pgsql php83-session php83-json php83-openssl
apk add labwc labwc-doc seatd dbus xwayland foot eudev
apk add font-dejavu font-noto font-awesome
apk add epiphany



## Entorno Grafico
rc-update add seatd default
rc-service seatd start
addgroup luicasad video
addgroup luicasad input
adduser luicasad seat
rc-update add dbus
rc-service dbus start

rc-update add udev
rc-update add udev-trigger
rc-update add udev-settle
rc-service udev start
udevadm trigger --action=add

add this lines to ~/.profile of luiscasad
```sh
export XDG_RUNTIME_DIR=/tmp/runtime-$(whoami)
mkdir -p -m 700 $XDG_RUNTIME_DIR
export WLR_DRM_NO_ATOMIC=1
export WLR_RENDERER=pixman
if [ -n "$SSH_CONNECTION" ]; then
        echo "conectado por SSH"
else
        dbus-run-session -- labwc
fi
```

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
                <!-- Teclado Windows: Super + B -->
                <keybind key="W-b">
                <action name="Execute" command="epiphany" />
        </keybind>

        <!-- Teclado Mac: Option + B (si Cmd se mapea como Alt en tu XKB model) -->
                <keybind key="A-b">
                <action name="Execute" command="epiphany" />
        </keybind>
        </keyboard>
</labwc_config>
```
I created the file ~/.config/labwc/environment with this configuration
```
XKB_DEFAULT_LAYOUT=us
```

I created the file ~/.config/labwc/autostart with this configuration
```
mako &
```

### menu
I created the file ~/.config/labwc/menu.xml with this configuration

```xml
<?xml version="1.0"?>
<openbox_menu>
  <menu id="root-menu" label="root-menu">
    <item label="Terminal">
      <action name="Execute" command="foot" />
    </item>
    <item label="Epiphany">
      <action name="Execute" command="epiphany" />
    </item>
    <item label="Reconfigure">
      <action name="Reconfigure" />
    </item>
    <item label="Exit">
      <action name="Exit" />
    </item>
  </menu>
</openbox_menu>
```




+ 5.- copy 42 ssh keys from host machine into VM
I opened a ssh session from host to VM in a terminal i call `remote` to help me wiht this explanation.

Inside a `local` terminal in Host machine I copied `~/.ssh/id_rsa`'s content.
In the `remote` terminal, with Vim, after setting mouse wiht `set mouse=n` y pasted the key.

Repeat for `id_rsa.pub`.

+ 6 Allow sshd forwarding port in /etc/ssh/sshd_config. It is required to edit from outside the virtual machine with Visual Studio Code and `Remote - ssh` extension.

```
AllowTcpForwarding yes
```








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
create user luicasad with encrypted password 'mysecretpassword';
create database piscineds;
grant all privileges on database piscineds to luicasad;
grant all provileges on schema public to luicasad;
```

create for user luicasad `~/.pgpass` file with permission 400

127.0.0.1:5432:piscineds:luicasad:mysecretpassword



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

#### Download adminer as luicasad

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
rc-update add adminer
``` 

