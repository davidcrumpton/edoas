# Enhanced DOAS

This is a proof-of-concept demonstration and should not be used in production. This adds option **l** like sudo which prints out rules defined for your user and option **m** to show what permitted rule was used to execute your command.

## Notes


1. The doas.conf file is safe to be world readble so this isn't necessary but it allows you to remove world readable perms from the file.
2. A user can use `doas -C /etc/doas.conf <cmd> <args> ` which will check if a command can be used but this is a guessing game.


## Security

This is a large change to doas and **has not** gone through the OpenBSD team scrutiny so use only in lab settings.  Install as /usr/local/bin/edoas without setuid set.

# Test Run

## Option l
Lists permit and deny rules available to the invoking user.  You can see I have superfluous run as root rules and only one is needed.

$ `doas -l`

```
Commands for user david (1000):
  permit group: wsrc (as root) ALL commands [ nopass setenv ] { FTPMODE PKG_CACHE PKG_PATH SM_PATH SSH_AUTH_SOCK DESTDIR DISTDIR FETCH_CMD FLAVOR GROUP MAKE MAKECONF MULTI_PACKAGES NOMAN OKAY_FILES OWNER PKG_DBDIR PKG_DESTDIR PKG_TMPDIR PORTSDIR RELEASEDIR SHARED_ONLY SUBPACKAGE WRKOBJDIR SUDO_PORT_V1 }
  permit group: wheel (as root) ALL commands [ keepenv persist ]
  permit user: david (as root) command: /usr/sbin/service apache2 restart
  permit user: david (as dcrumpton) command: /bin/ls /tmp
```

## Audit another user 
Here, admin david lists what user dcrumpton has permission to run.  User david is granted doas permission to run as user dcrumpton (wheel).

Lists permit and deny for run as user specified in u option.

$ `doas -u dcrumpton doas -l`

```
doas (david@intbsd.intbsd) password: 
Commands for user dcrumpton (1003):
    deny user: dcrumpton (as operator) command: /usr/sbin/rcctl restart apache2
  permit user: dcrumpton (as root) command: /usr/sbin/rcctl restart nginx
  permit user: dcrumpton (as operator) command: /usr/local/bin/backup_script [ nopass setenv ] { ZSH2=$ZSH -FOO BAR=high }
```

## Auditing run as settings for a different user
Admin david can see what user dcrumpton can run as operator

$ `doas -u operator doas -l`

```
Commands for user operator (2):
  permit user: operator (as root) command: rcctl reload httpd [ nopass ]
```


## Check permit rule used to execute command
What permit rule matched my command?


$ `doas -m id`

```
  permit group: wheel (as root) ALL commands [ keepenv persist ]
uid=0(root) gid=0(wheel) groups=0(wheel), 2(kmem), 3(sys), 4(tty), 5(operator), 20(staff), 31(guest)
```

## No rules defined
Lets you know there are no defined rules.


$ `doas -u certbot doas -l`

No allowed commands for user certbot (1002)


## Combinations
Can't be combined with option s


$ `doas -sl`

usage: doas [-Llmns] [-a style] [-C config] [-u user] command [arg ...]

