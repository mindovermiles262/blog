---
title: Join an Ubuntu 18 Server to an AD Domain
date: 2019-01-25
layout: post
author: andrew
image: assets/images/windowslinux.jpg
categories: [ linux, windows, "system administration" ]
---

`realmd` is an "on demand system DBus service which allows callers to configure network authentication and domain membership".  It is used to allow users to log into a Linux server using their AD domain credentials

`realmd` is a wrapper that configures `SSSD` or `Winbind` behind the scenes to do the actual network authentication and user account lookups.


## Adding Ubuntu Server to AD

We will be joining an Ubuntu 18.10 server to a Windows 2016 AD Domain.

* Ubuntu Server => `192.168.5.12`
** Hostname: `UBUNTU-1810`
* Windows 2016 AD Server => `192.168.5.50`
** Domain: `WILDERNESS.NET`
** Hostname: `YOSEMITE`

Additionally we will be setting up some security features so that only Domain Admins are able to log in. These Domain Admins will also have `sudo` privileges 

## Setup

All of the following are done under the root account. Feel freel to use `sudo` for every command if that makes you feel better.

### Install Packages

 apt-get install realmd sssd sssd-tools samba krb5-user packagekit adcli ntp

When prompted add your Kerberos version 5 realm (`WILDERNESS.NET`), your Kerberos Servers, and administrative server (`YOSEMITE.WILDERNESS.NET`) to the krb5-user config

Some of these packages are located in Ubuntu's `universe` repository. You may need to enable this repo if it says packages cannot be found.

### Configure NTP

Open `/etc/ntp.conf` and add the following line:

```
[ ... ]

# Specify one or more NTP servers.
server YOSEMITE.WILDERNESS.NET

[ ... ]
```

Then restart NTP, `systemctl restart ntp`

### Configure realmd

Next we will configure realmd. __Create__ `/etc/realmd.conf` and add the following:

```
[users]
    default-home = /home/%U
    default-shell = /bin/bash
[active-directory]
    default-client = sssd
    os-name = Ubuntu Server
    os-version = 18.10
[service]
    automatic-install = no
[wilderness.net]
    fully-qualified-names = no
    automatic-id-mapping = yes
    user-principal = yes
    manage-system = no
```

See Also: [realmd Conf Manual](https://freedesktop.org/software/realmd/docs/realmd-conf.html)

### Modify Kerberos Config

Lastly, we need to slightly modify the Kerberos config file. Open `/etc/krb5.conf` and edit the beginning lines to the following:

```
[libdefaults]
    default_realm = WILDERNESS.NET
```

## Join Server to AD Domain

Once set up, joining the server to the domain should be simple.

```
$ sudo systemctl restart ntp realmd

$ sudo kinit administrator@WILDERNESS.NET
Password for administrator@WILDERNESS.NET:
```

Then Join the domain:

```
$ sudo realm --verbose join WILDERNESS.NET --user-principal=UBUNTU/administrator@WILDERNESS.NET

[ ... ]

* Successfully enrolled machine in realm
```

Note: You must join the domain with a `Domain Administrator` account


## Setup User Logins

Now that we're joing to the domain, we need to allow users to log into the server using their AD accounts. Since they won't have a HOME directory until they log in, we will specifiy where to create these new folders and files.

### Configure Home Directories

In order for the home directory of a new user to be created, we need to edit the `/etc/pam.d/common-session` file to include:

```
[ ... ]

# Create new user home dir
session required pam_mkhomedir.so skel=/etc/skel/ umask=0077

#end of pam-auth-update config
```

### Configure SSDD

There are just a few tweaks needed in `/etc/sssd/sssd.conf` file:

1. Edit `access_provider` from `simple` to `ad` to allow SSH login

2. Add `ad_gpo_access_control` for RDP login

Once tweaked, restart sssd


## Login with Domain Credentials

Everything should now be set up to log in with your AD credentials. Try it out:

```
Ubuntu 18.10 ubuntu-1810 tty1

ubuntu-1810 login: rachel.carson@WILDERNESS.NET
Password:

[ Successful Login ]
```


## Setting User Permissions

### Allowing/Restricting Login Access

To enable/disable login by specific AD Security Groups use the CLI to edit realm:

```
# Permits ONLY Domain Admins
$ realm permit -g domain\ admins@WILDERNESS.NET
```

### sudo Access

To allow Domain Admins `sudo` rights edit the `/etc/sudoers`:

```
$ sudo visudo
```

Then add the line:

```
[ ... ]
%sudo  ALL=(ALL:ALL) ALL

# Grant Domain Admins sudo access
%domain\ admins@WILDERNESS.NET  ALL=(ALL:ALL) ALL

[ ... ]
```

Save and close.

## Troubleshooting

### "Resource Temporarily unavailable"

```
 $ sudo kinit admin@wilderness.net
 kinit: Resource Temporarily unavailable while getting initial credentials
```

__Problem:__ Your linux server cannot resolve your Kerberos or Administration (DC) Server.

__Fix:__
* Add the correct DNS server to your `/etc/resolv.conf` (Or edit your hosts file) to correctly resolve.
* Check which address is being accessed inside `/etc/kbr5.conf`

```
[realms]
  WILDERNESS.NET = {
    kdc = YOSEMITE.WILDERNESS.NET              # These are the server addresses
    admin_server = YOSEMITE.WILDERNESS.NET     # being checked
  }
```

### KDC Reply Did Not Match

```
$ sudo kinit user@WILDERNESS.NET
Password for user@WILDERNESS.NET: 
kinit: KDC reply did not match expectations while getting initial credentials
```

__Problem:__ Your domain needs to be upper-case in both the `kinit` login and inside your `/etc/krb5.conf`

__Fix:__ Change your `/etc/krb5.conf`'s default realm to use all caps:

```
[libdefaults]
  default_realm = WILDERNESS.NET 
```

## Further Reading

* [RHEL Windows Integration Guide PDF](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/pdf/Windows_Integration_Guide/Red_Hat_Enterprise_Linux-7-Windows_Integration_Guide-en-US.pdf)
* [Guide - Ubuntu 16 with AD Connectivity](http://ricktbaker.com/2017/11/08/ubuntu-16-with-active-directory-connectivity/)
