---
title: "Arch-ARM: xfce4 and x11vnc installation"
date:   2018-06-01 00:18:00 -0600
header:
    teaser: "{{ '/assets/img/posts/2018-06-01-arch-arm-xfce4-tigervnc/header.png' | absolute_url }}"
---

Before install [cXFCE] and [TigerVNC], I recommend read both config post and put attention on Installation and Configuration sections. This installation is only thought VNC connection, without HDMI display.

XFCE installation:
------

For XFCE works correctly, we need install two stancies indispensably: xfce4 and xfce4-goodies.

```bash
[alarm@192.168.1.XX ~]: sudo pacman -S xfce4 xfce4-goodies
:: There are 16 members in group xfce4:
:: Repository extra
   1) exo  2) garcon  3) gtk-xfce-engine  4) thunar  5) thunar-volman  6) tumbler  7) xfce4-appfinder  8) xfce4-panel
   9) xfce4-power-manager  10) xfce4-session  11) xfce4-settings  12) xfce4-terminal  13) xfconf  14) xfdesktop
   15) xfwm4  16) xfwm4-themes

Enter a selection (default=all): ''ENTER''
...
   32) xfce4-wavelan-plugin  33) xfce4-weather-plugin  34) xfce4-xkb-plugin
:: Repository community
   35) parole  36) ristretto  37) xfce4-whiskermenu-plugin

Enter a selection (default=all): ''ENTER''
...
:: Proceed with installation? [Y/n] y
```

After install XFCE4, you can start XFCE with typing on terminal (IMPORTANT: with this method, you need have a monitor connected!):

```bash
[alarm@192.168.1.XX ~]: startxfce4
```

TigerVNC installation:
------

```bash
[alarm@192.168.1.XX ~]: sudo pacman -S tigervnc
...
:: Proceed with installation? [Y/n] y
...
(3/4) Arming ConditionNeedsUpdate...
(4/4) Updating the desktop file MIME type cache...
```

After installation, we need type on terminal: vncserver With that VNC execute for first time and you need put a new password fot VNC connection. Also, vncserver create some config files:

```bash
[alarm@192.168.1.XX ~]: vncserver
You will require a password to access your desktops.

Password:
Verify:

Would you like to enter a view-only password (y/n)? n

xauth:  file /home/alarm/.Xauthority does not exist

New 'alarmpi:1 (alarm)' desktop is alarmpi:1

Creating default startup script /home/alarm/.vnc/xstartup
Creating default config /home/alarm/.vnc/config
Starting applications specified in /home/alarm/.vnc/xstartup
Log file is /home/alarm/.vnc/alarmpi:1.log
```

Now, kill the server for edit default startup script:

```bash
[alarm@192.168.1.XX ~]: vncserver -kill :1
Killing Xvnc process ID 0000X
```

Edit ~/.vnc/xstartup and add XFCE by default desktop environment:

> Remember make xstartup a execute file!

```bash
[alarm@192.168.1.XX ~]: cp ~/.vnc/xstartup ~/.vnc/xstartup.bckp
[alarm@192.168.1.XX ~]: nano ~/.vnc/xstartup
#!/bin/sh

unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
exec startxfce4
[alarm@192.168.1.XX ~]: chmod u+x ~/.vnc/xstartup
```

> Why create a copy of xstartup -> xstartup.bckp? Well, it's a good practice make a copy of any config file before modify it! So, if you need back to old config, only need rename that file!

Now, edit ~/.vnc/config Choose a screen size and other parameters if you need it:

```bash
[alarm@192.168.1.XX ~]: cp ~/.vnc/config ~/.vnc/config.bckp
[alarm@192.168.1.XX ~]: nano ~/.vnc/config
## Supported server options to pass to vncserver upon invocation can be listed
## in this file. See the following manpages for more: vncserver(1) Xvnc(1).
## Several common ones are shown below. Uncomment and modify to your liking.
##
securitytypes=vncauth
# desktop=sandbox
geometry=720x480
# localhost
# alwaysshared
```

> If you want some doubts about video screen size, [check this wiki post.]

After that, create a environment variable for a display:

```bash
[alarm@192.168.1.XX ~]: sudo nano /etc/profile
...
export DISPLAY=:0
[alarm@192.168.1.XX ~]: reboot
```

VNCserver it's ready!

> Remember, you need execute in terminal vncserver every time with need you!

To execute VNCserver:

```bash
[alarm@192.168.1.XX ~]: vncserver


New 'alarmpi:1 (alarm)' desktop is alarmpi:1

Starting applications specified in /home/alarm/.vnc/xstartup
Log file is /home/alarm/.vnc/alarmpi:1.log
```


To list all VNCserver:

```bash
[alarm@192.168.1.XX ~]: vncserver -list

TigerVNC server sessions:

X DISPLAY #     PROCESS ID
:1              329
```

To kill VNCserver:


```bash
[alarm@192.168.1.XX ~]: vncserver -kill :1
Killing Xvnc process ID 329
```

How to connect Raspberry VNCserver to a VNC viewer?:
------

First to all, we need know somethings. Every VNCserver by start with Raspberry generate a process with a display number. That's number it's interpreter by :X Where X it's the display number.

> Yep! You can generate VNC server's unlimited if you want! Only limitation it's Raspberry resources!

Every VNCserver generate a connection port 590X Again, X it's the number assign by VNCserver.
To connect to own Raspberry, with need execute vncserver on Raspberry side, and connect via VNCviewer by the port assign.
Every connection it's via Raspberry IP and port 590X.

Example:

> Raspberry side:

```bash
[alarm@192.168.1.XX ~]: vncserver


New 'alarmpi:1 (alarm)' desktop is alarmpi:1

Starting applications specified in /home/alarm/.vnc/xstartup
Log file is /home/alarm/.vnc/alarmpi:1.log
```

> VNCviewer side:

![VNCviewer Windows]({{"/assets/img/posts/2018-06-01-arch-arm-xfce4-tigervnc/001-vnc-viewer.png" | absolute_url }})

[You can download VNCviewer from here!]

Final result:

![VNCviewer]({{"/assets/img/posts/2018-06-01-arch-arm-xfce4-tigervnc/002-vnc-viewer.png" | absolute_url }})

If you want know the packages installed on my Raspberry, you can check this gist!

[TigerVNC]: https://wiki.archlinux.org/index.php/Tigervnc
[XFCE]: https://wiki.archlinux.org/index.php/xfce
[check this wiki post.]: https://en.wikipedia.org/wiki/File:Vector_Video_Standards8.svg
[You can download VNCviewer from here!]: https://www.realvnc.com/fr/connect/download/viewer/
[you can check this gist!]: https://gist.github.com/martinsgmx/b453474a6e30ddb852e6b971375cf8ad