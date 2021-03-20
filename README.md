# install_netbsd
NetBSD installation and first configuration

## Installation steps


## First steps with root

### Set up PKG URL
```sh
echo "PKG_PATH=ftp://ftp.NetBSD.org/pub/pkgsrc/packages/$(uname)/$(uname -m)/$(uname -r)/All/" >> /etc/pkg_install.conf
```

### Set up CPU frequency scaling:
Add to **/etc/rc.conf**:
```
estd="yes"
estd_flags=""
```

```sh
pkg_add estd
cp /usr/pkg/share/examples/rc.d/estd /etc/rc.d/
/etc/rc.d/estd restart
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

### Install pkgin
```sh
sudo pkg_add pkgin
```

Modify **/usr/pkg/etc/pkgin/repositories.conf**:
```
https://cdn.netbsd.org/pub/pkgsrc/packages/NetBSD/$arch/9.0/All
```
```
https://ftp.netbsd.org/pub/pkgsrc/packages/NetBSD/$arch/9.1/All
```
```sh
sudo pkgin update
sudo pkgin upgrade
```

### Install necessary packages:
```sh
sudo pkg_add bash bash_completion git icewm firefox emacs
```

```sh
sudo usermod -s /usr/pkg/bin/bash username
```

Create **~/.gitconfig**:
```
[user]
	name = <username>
	email = <email>
[pull]
	rebase = false
```

Create **~/.xsession**:
```
#!/bin/sh

xset -b
setxkbmap hu
exec icewm-session
```
```sh
chmod +x .xsession
```

Create **~/.Xdefaults**:
```
XTerm*background: black
XTerm*foreground: grey
XTerm*faceName: DejaVu Sans Mono
XTerm*faceSize: 11
XTerm*metaSendsEscape: true
```

Create **~/.bashrc**:
```
if [[ $- != *i* ]] ; then
  return
fi

export PAGER=less
export MANPAGER=less

export EDITOR=vim
export VISUAL=vim

HISTCONTROL=ignoreboth
shopt -s histappend

HISTSIZE=10000
HISTFILESIZE=2000

PS1='\[\033[01;31m\]\u\[\033[01;33m\]@\[\033[01;36m\]\h \[\033[01;33m\]\w \[\033[01;35m\]\$ \[\033[00m\]'

if [[ -f /usr/local/share/bash-completion/bash_completion.sh ]]; then
    source /usr/local/share/bash-completion/bash_completion.sh
fi
```

Create **~/.icewm/toolbar**:
```
prog XTerm xterm xterm +sb
prog Emacs emacs emacs
prog Firefox firefox firefox
```

Create **~/.icewm/preferences**:
```
OpaqueMove=0
OpaqueResize=0
TaskBarShowAPMStatus=0
TaskBarShowAPMAuto=1
TaskBarShowAPMTime=1
TaskBarShowAPMGraph=0
TaskBarShowCPUStatus=1
CPUStatusShowRamUsage=1
CPUStatusShowSwapUsage=1
CPUStatusShowAcpiTemp=1
CPUStatusShowAcpiTempInGraph=1
CPUStatusShowCpuFreq=1
TaskBarShowNetStatus=1
TaskBarCPUDelay=500
TaskBarCPUSamples=50
TaskBarMEMSamples=50
TaskBarMEMDelay=500
TaskBarNetSamples=50
TaskBarNetDelay=500
TaskBarApmGraphWidth=50
NetworkStatusDevice="wm0 ath0"
TimeFormat="%H:%M"
DateFormat="%a %Y.%m.%d."
WorkspaceNames=" 1 ", " 2 ", " 3 ", " 4 "
ShutdownCommand="sudo /sbin/shutdown -p now"
RebootCommand="sudo /sbin/shutdown -r now"
SuspendCommand="/usr/sbin/zzz"
```
