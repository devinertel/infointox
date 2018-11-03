---
id: 13
title: OpenVPN on NSLU2
date: 2005-12-18T16:57:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=13
permalink: /?p=13
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2005/12/openvpn-on-nslu2_18.html
categories:
  - Uncategorized
tags:
  - Linux
  - OpenVPN
---
Well I finally picked up a NSLU2 by Linksys. Have to say I am pretty impressed so far. A little device that fits in my hand just replaced 2 of my boxes at home. One for my fileserver and one for my openvpn server. Maybe at a later date I will go about how to flash it with unslung. But it is pretty easy so this doc is just for me to remember how I set up my openvpn.

So first thing is first go buy the NSLU2 and download unslung and flash it.
  
<http://www.nslu2-linux.org/>

Now you have it flashed. I put on a few packages first to allow me to work with it a bit better.
  
OpenSSH
  
Vim
  
Bash
  
Grep

Ok Now we are ready.
  
SSH to your NSLU2. If you have not set your password yet the default password is &#8220;uNSLUng&#8221;

Install OpenVPN:
  
**ipkg update;ipkg -force-depends install openvpn** 

Create Tun:
  
**mkdir /dev/net**
  
**mknod /dev/net/tun c 10 200**

Install Tun:
  
**insmod tun**

Enable Routing:
  
**echo 1 > /proc/sys/net/ipv4/ip_forward**

Now its time to generate our certs. I just downloaded openvpn on to my machine to create them. Goes much faster this way
  
You can download the current openvpn version here.
  
<http://openvpn.net/>
  
Also more details on this process can be found here.
  
<http://openvpn.net/howto.html#pki>

CD into the easy-rsa directory and edit the vars file with your information.

**. ./vars**
  
**./clean-all**
  
**./build-ca**

Now that the CA is up, we can build the keys for the server.

**./build-key-server server**

Now we have to build our client certs. I will only be buildling it for one client. I also use password protected certs.

**./build-key-pass client1**

Note: If you wanted other clients repeat the step with client2(or whatever you like). Remember to always use a unique common name for each client.

Generate Diffie Hellman parameters.
  
**./build-dh**

Now we need to create a direcotry on the NSLU2 to copy our keys to.
  
**mkdir -p /opt/etc/openvpn/keys**

You can copy these files to the NSLU2, may be a bit different for you:
  
**ca.crt, ca.key, dh1024.pem, server.crt, server.key, 01.pem, 02.pem, 03.pem, and 04.pem**

Now lets create a server.conf on the NSLU2 and write our conf file.
  
You can get a sample conf file from the previous download. I will just touch on the main things I change below.

I use TCP so I can proxy:
  
**\# TCP or UDP server?
  
proto tcp
  
;proto udp**

Choose a cipher of your choice. Must be the same on the client.
  
**\# Select a cryptographic cipher.
  
\# This config item must be copied to
  
\# the client config file as well.
  
;cipher BF-CBC # Blowfish (default)
  
;cipher AES-128-CBC # AES
  
;cipher DES-EDE3-CBC # Triple-DES**

Make sure it switches to priv nobody.
  
**\# You can uncomment this out on
  
\# non-Windows systems.
  
user nobody
  
group nobody**

Thats pretty much it for the server side.

For the client side. Its pretty much straight forward.Just make sure you have the right certs.
  
**ca.crt
  
client1.crt
  
client1.key**

Now back to the NSLU2.

For a while I have been trying to get MASQUERADE in iptables to work. But since the module is not in the ipkg repository and it is not enbaled in the kernel, this was not working. If you view the comment below Cooper did get this working and wrote a how-to for this. Since he wanted MASQUERADE for something a bit different I will document how I did it. I wanted it so I can hit other boxes behind the VPN. Without having to creat SSH tunnels (Which is what I was doing).
  
Here is Cooper&#8217;s doc on the NSLU site.

<http://www.nslu2-linux.org/wiki/HowTo/EnableIPMasquerading>

Now, for what I did for MASQUERADE.

First, I install the MASQUERADE modules, I used pre-compiled ones since I&#8217;m lazy. You can compile them yourself if you like, Cooper&#8217;s doc shows you how. Below is a link to pre-compiled ones.

<http://www.defector.de/docs/nslu2-ipmasq.htm>

Now you can install these.
  
**
  
ipkg-cl install kernel-module-ipt-masquerade\_2.4.22.l2.3r63-r7\_nslu2.ipk
  
ipkg-cl install kernel-module-ipt-state\_2.4.22.l2.3r63-r7\_nslu2.ipk
  
** 

Now lets install the modules.
  
**
  
insmod ip_tables
  
insmod iptable_filter
  
insmod ip_conntrack
  
insmod iptable_nat
  
insmod ipt_state
  
insmod ipt_MASQUERADE**

If some modules cannot be found, I may have forgot to document these when I was messing around with different modules.
  
You can easily find and install them. I actually don&#8217;t think you even need ipt\_state or iptable\_filter but I put them in there anyways to have a more full blown iptables.(in case of future work)

example:
  
ipkg list |grep conntrack

Now lets get all this stuff to run on reboot.

**
  
Create /opt/etc/init.d/S24openvpn and make it +x.**

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;"><span style="color: #666666; font-style: italic;">###################################################</span>
<span style="color: #666666; font-style: italic;">#!/bin/sh</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">&lt;</span>strong<span style="color: #000000; font-weight: bold;">&gt;</span> <span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#91;</span> <span style="color: #660033;">-n</span> <span style="color: #ff0000;">"<span style="color: #780078;">`pidof openvpn`</span>"</span> <span style="color: #7a0874; font-weight: bold;">&#93;</span>; <span style="color: #000000; font-weight: bold;">then</span>
<span style="color: #000000; font-weight: bold;">/</span>bin<span style="color: #000000; font-weight: bold;">/</span><span style="color: #c20cb9; font-weight: bold;">killall</span> openvpn <span style="color: #000000;">2</span><span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">/</span>dev<span style="color: #000000; font-weight: bold;">/</span>null
<span style="color: #000000; font-weight: bold;">fi</span><span style="color: #000000; font-weight: bold;">&lt;/</span>strong<span style="color: #000000; font-weight: bold;">&gt;</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">&lt;</span>strong<span style="color: #000000; font-weight: bold;">&gt;</span> <span style="color: #666666; font-style: italic;"># load kernel modules</span>
<span style="color: #000000; font-weight: bold;">/</span>sbin<span style="color: #000000; font-weight: bold;">/</span>insmod tun
<span style="color: #000000; font-weight: bold;">/</span>sbin<span style="color: #000000; font-weight: bold;">/</span>insmod ip_tables
<span style="color: #000000; font-weight: bold;">/</span>sbin<span style="color: #000000; font-weight: bold;">/</span>insmod iptable_filter
<span style="color: #000000; font-weight: bold;">/</span>sbin<span style="color: #000000; font-weight: bold;">/</span>insmod ip_conntrack
<span style="color: #000000; font-weight: bold;">/</span>sbin<span style="color: #000000; font-weight: bold;">/</span>insmod iptable_nat
<span style="color: #000000; font-weight: bold;">/</span>sbin<span style="color: #000000; font-weight: bold;">/</span>insmod ipt_state
<span style="color: #000000; font-weight: bold;">/</span>sbin<span style="color: #000000; font-weight: bold;">/</span>insmod ipt_MASQUERADE<span style="color: #000000; font-weight: bold;">&lt;/</span>strong<span style="color: #000000; font-weight: bold;">&gt;</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">&lt;</span>strong<span style="color: #000000; font-weight: bold;">&gt;</span> <span style="color: #666666; font-style: italic;"># enable IP forwarding</span>
<span style="color: #7a0874; font-weight: bold;">echo</span> <span style="color: #000000;">1</span> <span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000; font-weight: bold;">/</span>proc<span style="color: #000000; font-weight: bold;">/</span>sys<span style="color: #000000; font-weight: bold;">/</span>net<span style="color: #000000; font-weight: bold;">/</span>ipv4<span style="color: #000000; font-weight: bold;">/</span>ip_forward<span style="color: #000000; font-weight: bold;">&lt;/</span>strong<span style="color: #000000; font-weight: bold;">&gt;</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">&lt;</span>strong<span style="color: #000000; font-weight: bold;">&gt;</span> <span style="color: #666666; font-style: italic;"># set iptables rule</span>
iptables <span style="color: #660033;">-t</span> nat <span style="color: #660033;">-A</span> POSTROUTING <span style="color: #660033;">-s</span> 10.8.0.0<span style="color: #000000; font-weight: bold;">/</span><span style="color: #000000;">24</span> <span style="color: #660033;">-o</span> ixp0 <span style="color: #660033;">-j</span> MASQUERADE<span style="color: #000000; font-weight: bold;">&lt;/</span>strong<span style="color: #000000; font-weight: bold;">&gt;</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">&lt;</span>strong<span style="color: #000000; font-weight: bold;">&gt;</span> <span style="color: #666666; font-style: italic;"># Startup VPN tunnel in daemon mode</span>
<span style="color: #000000; font-weight: bold;">/</span>opt<span style="color: #000000; font-weight: bold;">/</span>sbin<span style="color: #000000; font-weight: bold;">/</span>openvpn <span style="color: #660033;">--cd</span> <span style="color: #000000; font-weight: bold;">/</span>opt<span style="color: #000000; font-weight: bold;">/</span>etc<span style="color: #000000; font-weight: bold;">/</span>openvpn <span style="color: #660033;">--daemon</span> \
<span style="color: #660033;">--log-append</span> <span style="color: #000000; font-weight: bold;">/</span>var<span style="color: #000000; font-weight: bold;">/</span>log<span style="color: #000000; font-weight: bold;">/</span>openvpn.log \
<span style="color: #660033;">--config</span> server.conf
<span style="color: #666666; font-style: italic;">###################################################</span></pre>
      </td>
    </tr>
  </table>
</div>

Now you are all set! Test it out !

Thanks to everyone!