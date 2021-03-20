# install_netbsd
NetBSD installation and first configuration

## Installation steps


## First steps with root

### Set up PKG URL
```sh
echo "PKG_PATH=ftp://ftp.NetBSD.org/pub/pkgsrc/packages/$(uname)/$(uname -m)/$(uname -r)/All/" >> /etc/pkg_install.conf
```

### Set up CPU frequency scaling:
Add to **/etc/rc.conf**
```
estd="yes"
estd_flags=""
```

```sh
pkg_add estd
cp /usr/pkg/share/examples/rc.d/estd /etc/rc.d/
/etc/rc.d/estd restart
```

### Install `bash`:
```sh
pkg_add bash
usermod -s /usr/pkg/bin/bash username
```

### Install `sudo`:
```sh
pkg_add sudo
visudo
```
Remove comment, to let **wheel** group use `sudo`:
```
## Uncomment to allow members of group wheel to execute any command
# %wheel ALL=(ALL) ALL
```
```
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL) ALL
```

## First steps with user

### Install necessary packages:
```sh
sudo pkg_add icewm firefox
```
