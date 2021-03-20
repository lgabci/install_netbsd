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

### Set up WPA

Install `wpa_supplicant`:
```sh
sudo pkd_add wpa_supplicant
```

Modify **/etc/wpa_supplicant.conf**:
```
# $NetBSD: wpa_supplicant.conf,v 1.1 2019/01/12 16:51:54 roy Exp $

# Allow wpa_cli(8) to configure wpa_supplicant
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=wheel
update_config=1
```
```
ctrl_interface=/var/run/wpa_supplicant
ctrl_interface_group=wheel
update_config=0
ap_scan=1

network={
	ssid="<SSID0>"
	scan_ssid=0
	priority=10
	proto=RSN
	key_mgmt=WPA-PSK
	pairwise=CCMP
	group=CCMP
	psk="<PSK0>"
}

network={
	ssid="<SSID1>"
	scan_ssid=0
	priority=5
	proto=RSN
	key_mgmt=WPA-PSK
	pairwise=CCMP
	group=CCMP
	psk="<PSK1>"
}
```

Add to **/etc/rc.conf**:
```
wpa_supplicant=YES
wpa_supplicant_flags="-B -i <NIC> -c /etc/wpa_supplicant.conf"
dhcpcd=YES
dhcpcd_flags="-q -b <NIC> <NIC>"
```

### Install necessary packages:
```sh
sudo pkg_add bash bash_completion git icewm firefox emacs vim
```

Set `bash` for *username*'s shell
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

if [ -r /usr/pkg/share/bash-completion/bash_completion ]; then
  . /usr/pkg/share/bash-completion/bash_completion
fi
```

Create **~/.icewm/toolbar**:
```
prog XTerm xterm xterm +sb
prog Emacs emacs emacs
prog Firefox firefox firefox
```

Create **~/.icewm/menu**:
```
include /usr/pkg/share/icewm/menu

separator
prog KiCad kicad kicad
prog DOSBox dosbox dosbox
prog "VLC media player" vlc vlc
prog Xcalc accessories-calculator xcalc
```

Create **~/.icewm/theme**:
```
Theme="metal2/default.theme"
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
Create **~/.emacs.d/init.el**:
```lisp
;; load theme
(load-theme 'manoj-dark)

;; hide toolbar
(tool-bar-mode -1)

;; show current row and column
(setq column-number-mode t)

;; show matching parentheses
(show-paren-mode 1)

;; show characters over 80 columns
(require 'whitespace)
(setq whitespace-line-column 80)
(setq whitespace-style '(face lines-tail tab-mark trailing))
(global-whitespace-mode +1)

;; hihglight TODO-s
(add-hook 'prog-mode-hook
          (lambda ()
            (font-lock-add-keywords nil
              '(("\\(\\<\\(FIXME\\|TODO\\|BUG\\)[: \n]\\|#\\#.*\\|/\\/.*\\)" 0
              font-lock-warning-face t)))))

;; turn off indent tabs mode
(setq-default indent-tabs-mode nil)

;; set font size
(set-face-attribute 'default nil :height 90)

;; save desktop
(setq desktop-auto-save-timeout nil)
(setq desktop-restore-forces-onscreen nil)
(add-hook 'desktop-after-read-hook
          (lambda()
            (frameset-restore
              desktop-saved-frameset
              :reuse-frames (eq desktop-restore-reuses-frames t)
              :cleanup-frames (not (eq desktop-restore-reuses-frames 'keep))
              :force-display desktop-restore-in-current-display
              :force-onscreen desktop-restore-forces-onscreen)))
(desktop-save-mode 1)

;; auto refresh modified buffers
(global-auto-revert-mode t)

;; autosaves and backups in /tmp
(setq backup-directory-alist
      `((".*" . ,temporary-file-directory)))
(setq auto-save-file-name-transforms
      `((".*" ,temporary-file-directory t)))

;; disable lock files
(setq create-lockfiles nil)

;; set C style
(setq c-default-style
      '((java-mode . "java")
        (awk-mode . "awk")
        (other . "stroustrup")))
(setq-default c-basic-offset 2
              tab-width 8
              indent-tabs-mode nil)

;; function to set current window width to 80 columns
(defun set-window-width (n)
  "Set the selected window's width."
  (adjust-window-trailing-edge (selected-window) (- n (window-width)) t))

(defun set-80-columns ()
  "Set the selected window to 80 columns."
  (interactive)
  (set-window-width 80))

(global-set-key "\C-x~" 'set-80-columns)
```

Create **~/.emacs.d/init_bash.sh**:
```sh
. ~/.bashrc
```

Create **~/.vimrc**:
```
source /usr/pkg/share/vim/vim82/defaults.vim

set tabstop=8
set shiftwidth=2
set expandtab
```

### Set up **mdns**:

Add to **/etc/rc.conf**:
```
mdnsd=YES
```

Modify **/etc/nsswitch.conf**:
```
hosts:          files dns
```
```
hosts:          files dns mdnsd
```

### Enable `ntpdate`

Add to **/etc/rc.conf**:
```
ntpdate=YES
```

### Enable `xdm`

Add to **/etc/rc.conf**:
```
xdm=YES
```

### Set up `ssh`

Add to **/etc/rc.conf**:
```
sshd=YES
```

Add to **/etc/ssh/ssh_config**:
```

Host *.local
   CheckHostIP no
```

Modify **/etc/ssh/sshd_config**:
```
#PasswordAuthentication yes
```
```
PasswordAuthentication no
```
and
```
UsePam yes
```
```
UsePam no
```
