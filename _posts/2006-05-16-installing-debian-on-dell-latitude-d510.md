---
id: 18
title: Installing Debian on Dell Latitude D510
date: 2006-05-16T20:04:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=18
permalink: /?p=18
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2006/05/installing-debian-on-dell-latitude-d510_16.html
categories:
  - Uncategorized
tags:
  - Linux
---
Well just got one of these things from work. Kind of stinks because my IBM A31 played so nice with linux. So I&#8217;m writing this to document this long proces and hopefully someone else doesn&#8217;t have to reinstall a million times while pulling all their hair out.

First of all booting with the debian-installer with the option “linux26” will not work. These drives are SATA, and the kernel used under that option does not pick it up. So download the stable CD and install it with default options giving you the 2.4 kernel. The 2.4 kernel will pick up the sata as IDE. Ok now we have a base “stable” debian system.

**Step 1 (get testing packages):**
  
Add testing to your sources.list (I always add after main &#8211; contrib & non-free, that’s your call)

* apt-get update
  
* apt-get dist-upgrade &#8211; This is optional, you can stay on stable if you want or just do it later.

Note:
  
If you do the dist-upgrade use xserver-xorg for your xserver this has support for the video card “i810”
  
If you stay on Xfree86 you can use the “vesa” driver but you will not get direct rendering.

**Step 2 (install/boot 2.6 kernel):**
  
At the time of writing this, 2.6.15 was new and had built-in support for ipw2200, so that’s what I went with.

* apt-get install linux-image-2.6.15-1-686

Now we have to change some things to point to sda.

Change in /boot/grub/menu.lst
  
* search for kopt and change to sda
  
* look for the section the kernel boots and change to sda

Change /boot/grub/device.map
  
* change the hdc to sda

Change in /etc/fstab
  
* change the hd’s sd’s
  
* change the cdrom scd0 (this will be used later)

**Step 3 (Get scd0/CDROM working):** 
  
We will have to recompile the kernel for this one. So grab the source.
  
If you have not already you will need to install kernel-package for this way.
  
You can always comipile the kernel your own way too.

* apt-get install linux-source-2.6.15
  
* extract kernel in /usr/src and make ln –s linux kernel-source-dir
  
* now copy current kernel config from /boot/ to /usr/src/linux/.config

You can make other changes but I’m just showing the one for the sata cdrom.

* edit /usr/src/linux/drivers/scsi/libata-core.c and change int atapi_enabled = 0 to =1

Now we compile
  
* make oldconfig
  
* make-kpkg &#8211;initrd &#8211;append-to-version=&#8221;-devin\_ertel\_rocks&#8221; kernel_image <&#8211;hope you dont cp/paste :>

Now install it and boot it. Then mount a cd! You can get rid of your other kernels if you like.

**Step 4 (Built-in Wireless – IPW2200):**

Since ipw2200 is built in to 2.6.15 all we have to do is drop the firmware in /lib/firmware.
  
Or you just look in /etc/hotplug/firmware.agent to see where hotplug looks for firmware.

* Download the ipw2200 firmware (at the time I needed 2.4, maybe different now)

<http://ipw2200.sourceforge.net/>

* Extract it into /lib/firmware

**Step 5 (resolution and direct rendering):**

Now you will have to do a dist-upgrade for this to work correctly. Not sure what package gets direct rendering (xlibs, new xorg???) but this is the only way it worked for me.

Note:
  
It does get a better FPS in glxgears and looks a lot crisper but really does not seem to be the
  
resolution I set. So if anyone sees something I am doing wrong or has other ideas I would love to
  
hear them.

* apt-get install 915resolution (you can read more about what this does from the links below)
  
* set your /etc/defaults/915resolution &#8211; set how you like(below is how I did it)

MODE=3c
  
XRESO=1400
  
YRESO=1050
  
BIT=32

* /etc/init.d/915resolution start
  
* edit /etc/X11/xorg.conf
  
* change your driver to i810 if you haven’t already
  
* change your screen to 1400&#215;1050
  
* restart X

**Step 6 (Touchpad Driver/Synapatics):**
  
Well I never really liked touchpads, and with the default pointer driver it was terrible in enlightenment(not as bad in gnome).

* Install x-dev, libx11-dev and libxext-dev
  
* Download The Synapatics driver.
  
<http://freshmeat.net/projects/synaptics/>

* extract and make install
  
* then just follow the instructions in the INSTALL file.

Note:
  
I changed the speed, it was way to slow at first.
  
Also, for some reason I would loose the mouse after undocking. To fix this I changed the &#8220;ServerLayout&#8221; like so:

Section &#8220;ServerLayout&#8221;
  
InputDevice &#8220;Synaptics Mouse&#8221; &#8220;AlwaysCore&#8221;

* also make the change in the device section

**Step 7 (ACPID/lidbtn):**
  
Everytime I would close the lid, it would go either to sleep or standby and it would not recover. So to fix this I configured acpid to just blank the screen.
  
So add the files below to /etc/acpi. You will need vbetool, this is how I blank the screen.
  
There is a lot more you can do with acpid but this was the big one for me.

/etc/acpi/lidbtn.sh <&#8211;make this +x

&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;-
  
#!/bin/sh
  
\# /etc/acpi/lidbtn.sh
  
grep &#8216;open&#8217; /proc/acpi/button/lid/LID/state >/dev/null

if [ &#8220;x$?&#8221; == &#8220;x0&#8221; ]; then
  
/usr/sbin/vbetool dpms on
  
else
  
/usr/sbin/vbetool dpms off
  
fi
  
&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8211;

and

/etc/acpi/events/lidbtn

&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;-
  
#/etc/acpi/events/lidbtn

event=button[ /]lid
  
action=/etc/acpi/lidbtn.sh

&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;-

* Now restart acpid /etc/init.d/acpid restart

Other Things To Do:
  
* Still would like to get the LED for the wireless to work. I found when compiled into the kernel it was a little to buggy.

* Would like to get ipw2200 in monitor mode. I tried to compile this in,
  
CONFIG\_IPW2200\_MONITOR=y but it kept removing it. It seems like newer kernel versions in debian will be enabled. Just like the IPW2100. Any ideas , again would love to hear.

* Get the function keys working good. There is a package called i8kutils , but I have not had much luck with that. I may look into acpi to handle this.

References:
  
<http://ipw2200.sourceforge.net/>
  
<http://asl.epfl.ch/~kolski/d505.html>
  
<http://www-inf.int-evry.fr/~olberger/weblog/2005/08/24/debian-gnulinux-on-a-dell-latitude-d510-laptop/>
  
<http://alpha.uhasselt.be/Research/Algebra/Members/D505.html>
  
<http://perso.wanadoo.fr/apoirier/>
  
<http://lists.debian.org/debian-kernel/2006/03/msg00614.html>
  
<http://clx.digi.com.br/wiki/bin/view/Personal/DellLatitude110L>
  
<http://csd.informatik.uni-oldenburg.de/~eagle/acpid.html#sec-4>