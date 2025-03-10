# Alpine WAYLAND desktop setup: FF version
===========================================================

Alpine must be previously installed. This will install a new fashioned desktop, for more traditional check [../../newbie/alpine-newbie-xfce-desktop.md](../../newbie/alpine-newbie-xfce-desktop.md)

* [How to use this guide](#how-to-use-this-guide)
* [Preparation](#preparation-alpine)
    * [setup OS configuration](#setup-os-configuration)
    * [setup system users](#setup-system-users)
    * [setup hardware support](#setup-hardware-support)
    * [setup audio and video](#setup-audio-and-video)
* [Instalacion WAYLAND Alpine](#instalacion-wayland-apine)
    * [wayland sway console login](#wayland-sway-console-login)
    * [wayland gui login manager](#wayland-gui-login-manager)
    * [multimedia and device enhanced](#multimedia-and-device-enhanced)
* [Licensing clarifications](#licensing-clarifications)
* [See also](#see-also)

## preparation Alpine

You must have already installed alpine, and wayland only works well in alpine 3.14 and up

> **Warning** **YOU MUST HAVE DIRECT WIRED INTERNET, if not ask for an ISO from VenenuX:** [https://t.me/alpine_linux/762](https://t.me/s/alpine_linux/762)
or configure a network connection check [alpine-tutorial-wifi-routering.md](alpine-tutorial-wifi-routering.md)

#### setup OS configuration

Feels lost here? check [How to use this guide](#how-to-use-this-guide) section of this document


```
sed -i -r 's|#PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config

rc-service sshd restart;rc-update add sshd default

hostname venenux-desktop
echo 'hostname="venenux-desktop"' > /etc/conf.d/hostname 
echo "venenux-desktop" > /etc/hostname

cat > /etc/hosts << EOF
127.0.0.1 venenux-desktop localhost.localdomain localhost
::1 localhost localhost.localdomain
EOF

cat > /etc/network/interfaces << EOF
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet dhcp

iface eth0 inet6 dhcp
    pre-up echo 0 > /proc/sys/net/ipv6/conf/eth0/accept_ra
EOF

rc-service networking restart;rc-update add networking boot

cat > /root/.cshrc << EOF
unsetenv DISPLAY || true
HISTCONTROL=ignoreboth
EOF

cp /root/.cshrc  /root/.bashrc

 echo "root:toor" | chpasswd

apk add tcsh

add-shell '/bin/csh'

adduser -D -g "" -u 998 -h /opt/daru -s /bin/csh daru

 echo "daru:daru" | chpasswd

rm -f /opt/daru/*
mkdir /opt/daru
cat > /opt/daru/.cshrc << EOF
unsetenv DISPLAY
export PAGER=less
set autologout = 6
set prompt = "$ "
set history = 0
set ignoreeof
EOF
cp /opt/daru/.cshrc /opt/daru/.bashrc
chown -R daru:daru /opt/daru

cat > /etc/apk/repositories << EOF
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/main
http://dl-4.alpinelinux.org/alpine/v$(cat /etc/alpine-release | cut -d'.' -f1,2)/community
EOF

apk update

apk add man-db man-pages nano binutils coreutils readline \
 sed attr dialog lsof less groff wget curl terminus-font \
 zip p7zip xz tar cabextract cpio binutils lha acpi musl-locales musl-locales-lang \
 e2fsprogs e2fsprogs-doc btrfs-progs btrfs-progs-doc exfat-utils \
 f2fs-tools f2fs-tools-doc dosfstools dosfstools-doc xfsprogs xfsprogs-doc jfsutils jfsutils-doc \
 testdisk testdisk-doc partimage partimage-doc parted parted-doc util-linux zram-init

sed -i "s#.*consolefont.*=.*#consolefont="ter-132n.psf.gz"#g" /etc/conf.d/consolefont
setfont /usr/share/consolefonts/ter-132n.psf.gz

rc-update add consolefont boot
```

For more extended info check [../../newbie/alpine-newbie-xfce-desktop.md](../../newbie/alpine-newbie-xfce-desktop.md#setup-os-configuration)

#### setup system users

```
apk add shadow shadow-uidmap doas musl-locales musl-locales-lang

cat > /tmp/tmp.tmp << EOF
set history = 10000
set prompt = "$ "
EOF

mkdir /etc/skel
cat /tmp/tmp.tmp > /etc/skel/.cshrc
cat /tmp/tmp.tmp > /etc/skel/.bashrc

cat > /etc/skel/.Xresources << EOF
Xft.antialias: 0
Xft.rgba:      rgb
Xft.autohint:  0
Xft.hinting:   1
Xft.hintstyle: hintslight
EOF

cat > /etc/default/useradd << EOF
# useradd defaults file
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
EOF

cat > /etc/login.defs << EOF
USERGROUPS_ENAB yes
SYSLOG_SU_ENAB		yes
SYSLOG_SG_ENAB		yes
SULOG_FILE	/var/log/sulog
SU_NAME		su
EOF

useradd -m -U -c "" -G wheel,input,disk,floppy,cdrom,dialout,audio,video,lp,netdev,games,users,ping general

for u in $(ls /home); do for g in disk lp floppy audio cdrom dialout video lp netdev games users ping; do addgroup $u $g; done;done
```

> **Warning**  your user name must be `general`, you can put a "human name" as you wish, later.

For more details check  [../../newbie/alpine-newbie-configuration.md](../../newbie/alpine-newbie-configuration.md#setup-system-users)

#### setup hardware support

```
apk add acpi eudev eudev-doc eudev-rule-generator eudev-openrc pciutils util-linux zram-init \
 fuse fuse-exfat-utils fuse-exfat avfs pcre2 cpufreqd bluez bluez-openrc \
 wpa_supplicant dhcpcd chrony macchanger wireless-tools iputils linux-firmware \
 networkmanager networkmanager-lang networkmanager-openvpn networkmanager-openvpn-lang

rc-update add udev
rc-update add acpid
rc-update add cpufreqd
rc-update add fuse
rc-update add bluetooth
rc-update add chronyd
rc-update add wpa_supplicant
rc-update add networkmanager

rc-service networking restart

rc-service wpa_supplicant restart

rc-service bluetooth restart

rc-service udev restart 

rc-service fuse restart

rc-service cpufreqd restart

```

For more details check  [../../newbie/alpine-newbie-configuration.md](../../newbie/alpine-newbie-configuration.md#setup-software-graphical-fonts-and-languajes)

#### setup audio and video for wayland

> **Note** on alpine 3.14 gtk3 will force xorg dependencies.. for 3.16 will use gtk4 and SDL2

```
apk add xf86-input-evdev cairo pango pixman \
 mesa xinit mesa-dri-gallium xf86-video-modesetting xf86-input-libinput libxinerama xrandr kbd setxkbmap

apk add bluez bluez-openrc polkit polkit-openrc polkit-elogind udisks2 udisks2-lang \
 dbus dbus-x11 elogind elogind-openrc \
 gvfs gvfs-fuse gvfs-archive gvfs-dav gvfs-nfs gvfs-lang \
 networkmanager-elogind 

dbus-uuidgen > /var/lib/dbus/machine-id

rc-update add dbus
rc-update add elogind
rc-update add polkit

apk add font-noto-all ttf-dejavu ttf-linux-libertine ttf-liberation \
 font-bitstream-type1 font-bitstream-100dpi font-bitstream-75dpi \
 font-adobe-utopia-type1 font-adobe-utopia-75dpi font-adobe-utopia-100dpi \
 font-isas-misc

apk add alsa-utils alsa-plugins alsa-tools alsaconf \
 pipewire pipewire-pulse pipewire-alsa pipewire-spa-bluez

cat > /etc/security/limits.d/audio-limits.conf << EOF
@audio - memlock 256
@audio - nice -11
@audio - rtprio 88
EOF

rc-service dbus restart

rc-service elogind restart

rc-service polkit restart

rc-update del lightdm
```

> **Warning**  your user name must be `general`, you can put a "human name" as you wish, later.

> **Note** pure wayland will work only in modern gpu, otherwise will use xwayland if you need some modules like xf86-video packages as `xf86-video-intel`, `xf86-video-amdgpu`, `xf86-video-noveau`, `xf86-video-ati` or `xf86-video-nv`

## Instalacion WAYLAND Alpine

```
apk add wayland wlroots foot sway sway-doc bemenu swaylock swaylockd swaybg swayidle \
 weston weston-backend-wayland weston-backend-x11 weston-backend-drm weston-backend-wayland weston-backend-headless \
 weston-doc weston-shell-desktop weston-desktop-x11 weston-clients weston-terminal \
 weston-xwayland weston-shell-desktop weston-shell-fullscreen weston-cms-static
```

At this point you already has a waylan environment and can choose beetween weston and sway, 
just login into and start your desktop, weston is just the first implementation, can be run 
inside and X11 or another wayland session, sway is a window manager and compositor.

#### configurations

```
for u in $(ls /home); do mkdir -p /home/$u/.config/sway/ && cp /etc/sway/config /home/$u/.config/sway/config ;done

for u in $(ls /home); touch /home/$u/.config/weston.ini;done
```

The wayland weston and sway configurations depens on your preferences, 
the above commands just provide defaults to made those compositors able to run for users.

#### wayland sway console login

If want autologin with TTY use this script to your system users:

```
mkdir /home/general/

cat > /etc/skel/.profile << EOF
    if test -z "\${XDG_RUNTIME_DIR}"; then
      export XDG_RUNTIME_DIR=/tmp/\$(id -u)-runtime-dir
      if ! test -d "\${XDG_RUNTIME_DIR}"; then
        mkdir "\${XDG_RUNTIME_DIR}"
        chmod 0700 "\${XDG_RUNTIME_DIR}"
      fi
    fi
EOF
for u in $(ls /home); do cp /etc/skel/.profile /home/$u/ ;done

cat > /home/general/.xinitrc << EOF

if [ -z "\${DISPLAY}" ] && [ "\${XDG_VTNR}" -eq 1 ]; then
  if [ "\$(fgconsole 2>/dev/null || echo -1)" -eq 1 ]; then
    dbus-run-session -- sway
    export SWAYSOCK=/run/user/$(id -u)/sway-ipc.$(id -u).$(pgrep -x sway).sock
  fi
fi
EOF

```

#### wayland gui login manager


Wayland its on early stages.. so there is no login manager compatible, 
this is cos wayland per ser its another way to run GUI, and 
all the sesion and login GUI managers runs over Xorg.

```
apk add lightdm elogind elogind-openrc elogind-lang polkit polkit-openrc polkit-elogind \
 lightdm-lang lightdm-gtk-greeter 

rc-update add lightdm

rc-service lightdm restart
```

> **Warning** : for alpine 3.14, 3.15 just works the login sesion for sway, maybe 3.16 and up will 
result in a blank screen, check https://github.com/swaywm/sway/pull/3634#issuecomment-462779163

#### desktop integration and device media


```
apk add xdg-desktop-portal xdg-desktop-portal-wlr xdg-desktop-portal-lang xdg-desktop-portal-gtk xdg-desktop-portal-gtk-lang
```

#### multimedia and device enhanced

```
apk add gst-plugins-base gst-plugins-bad gst-plugins-ugly gst-plugins-good \
 libcanberra-gstreamer wxgtk-media \
 mediainfo ffmpeg ffmpeg-doc ffmpeg-libs lame lame-doc rtkit rtkit-doc \
 mpv mpv-doc deadbeef deadbeef-lang libxinerama xrandr 

for u in $(ls /home); do for g in plugdev audio cdrom dialout video netdev; do addgroup $u $g; done;done

cat > /etc/network/interfaces << EOF
auto lo
iface lo inet loopback
EOF

service networking restart

service wpa_supplicant restart

service networkmanager restart

```

## How to use this guide

This guide **structure all the commands in blocks, each block its separated by a line spaced**, 
so you must **type each line as is.. and hit enter**, so you noted that then you 
typed each separated clocks of commands, copy/type only blocks separated by an empty line, 
all new(next) lines are made by just enter. the terminal will detect if must execute or not.

**If you have another computer or gui**, try to use SSH client like putty or just in terminal (MAC or Linux) do:

1. at the Alpine installation: `sed -i 's|.*PermitRootLogin.*|PermitRootLogin yes|g' /etc/ssh/sshd_config;service sshd restart`
2. at the other OS just connect: `ssh -l root <ip>` change "`<ip>`" with the address of your device.
3. copy each separated by empty line, block of command, copy only blocks separate by empty line
4. and paste each separated by empty line block in the remnote (ssh), do not paste all the blocks at same time!

> **Warning** Some Linux or/and Mac terminals have security cut/paste locks, so 
if you paste, the first line will be preceded by garbage, check always the first char of your paste.

> **Warning** after finish, rerun: `sed -i -r 's|.*PermitRootLogin.*|PermitRootLogin no|g' /etc/ssh/sshd_config`
and restart ssh `service sshd restart` becouse security implications.

Done? return to [Preparation](#preparation-alpine) section of this document.

#### hardware used

| item             | minimal feature   | Extra recommendations              |
| ---------------- | ----------------- | ---------------------------------- |
| RAM MB           | 1Gb DDR1          | 6Gb DDR3, web browsers consumes so much |
| CPU              | intel Dual Core   | Not necesary                       |
| RAM CPU          | 2Mb (L2) 4kb/L1   |  |
| GPU              | intel G41         | Radeon X1200 For web browsers and modern apps will be need |
| RAM GPU          | 256Mb             | 1Gb For web browsers and modern apps will be need |
| Storage          | 120Gb HDD WD      | 256Gb SSD are mandatory for speed |
| ARCH             | 32bits (i386/arm6)| 64bits (amd64) mandatory for most modern apps unfortunatelly |
| Audio            | AC 97             | HD audio and HDMI audio are a mess |

#### usernames

| item      | name                | password |
| --------- | ------------------- | -------- |
| remote    | daru                | daru     |
| admin     | root                | toor     |
| user      | general             | general  |

Done? return to [Preparation](#preparation-alpine) section of this document.

## Licensing clarifications

**CC BY-NC-SA**: the project allows reusers to distribute, remix, adapt, and build upon the material 
in any medium or format for noncommercial purposes only, and only so long as attribution is given 
to the creators involved. If you remix, adapt, or build upon the material, you must license the modified 
material under identical terms,  includes the following elements:

* **BY**  – Credit must be given to the creator of each content respectivelly, starting at the first contributor.
* **NC**  – Only noncommercial uses of the work are permitted, with exceptions if you fill an issue here!
* **SA**  – Adaptations must be shared under the same terms, you must obey this terms and do not change it.

https://codeberg.org/alpine/alpine-wiki/src/branch/main#license

## See also

* [README.md](README.md)
* [alpine-newbie-install.md](../../newbie/alpine-newbie-install.md)
* [alpine-tutorial-desktop-xfce4-fast-forward.md](alpine-tutorial-desktop-xfce4-fast-forward.md)
