---
id: 19
title: Cracking WEP / WPA-PSK
date: 2006-07-28T14:44:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=19
permalink: /?p=19
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2006/07/cracking-wep-wpa-psk_28.html
categories:
  - Wifi
tags:
  - WEP
  - Wifi
---
Ok, I know there are tons of docs out there on this and it has been done a million times. This is just for my personal reference. I always knew WEP was insecure, I just never did anything about it (found it boring). So on one bored night I decided to find out how long it would take to break into my MAC Filtering/ WEP 128 Bit key network. It took about 1 hour to gather all the IV’s I needed and to crack the key. So here’s how to do it.

First you need aircrack.
  
<http://www.aircrack-ng.org>

I will usually find the network I want to attack using Kismet. Then let the fun begin.
  
Now we can startup airodump-ng to capture all the stuff we need.

**airodump-ng -w wepcrack -c 1 wlan0**

To save headaches of MAC filtering lets just spoof our MAC to a client that is already connected or one you know is allowed.
  
(If nobody is connected and MAC filtering is enabled, you are kind of out of luck)
  
**
  
ifconfig wlan0 down
  
ifconfig wlan0 hw ether FF:31:13:3F:44:55 (client MAC)
  
ifconfig wlan0 up**

There much better, MAC filtering is defeated.

Ok now we are capturing the data with airodump, lets inject some traffic and generate some IV’s. In airodump the data column is the IV’s. For a 64 bit key you need around 300,000 and about 1 million for 128 bit key. But this will vary. On to the injection.

Here is a common ARP-request replay attack, which works pretty well.

**Aireplay-ng –3 –b 00:14:BF:18:9F:88 (bssid of AP) -h FF:31:13:3F:44:55 (client) wlan0**

You may have to wait a while for the first ARP request to be seen, but once it gets a couple its all down hill from there.

Aireplay-ng will look like below when running.

Saving ARP requests in replay_arp-0727-12134.cap
  
You must also start airodump to capture replies.
  
Read 3643 packets (got 3 ARP requests), sent 2537 packets&#8230;

Note: If you cannot get any ARP requests, sometimes doing a de-auth on the client will sometimes generate some traffic for you. It is done like below.
  
(If you want to DOS the client just change the 20 to a 0, this will make it loop rather then run 20 times)

**Aireplay-ng –0 20 –a 00:14:BF:18:9F:88 (bssid of AP) –c FF:31:13:3F:44:55 (client) wlan0**

Now you have an ARP and are replaying traffic. Now just wait for the IV’s to come in.
  
Once you have enough IV’s lets crack the .cap file.

At a basic level you can just run it like below. By default it tries to crack a 128 bit key. Sometimes its best to start with a 64 bit key and work your way up. Its all up to you.

**aircrack-ng [options] capture_file**

Sometimes you will have to play with some options depending on the key. Please refer to aircrack’s site for more explanation. It is very straightforward.

<http://www.aircrack-ng.org/doku.php?id=aircrack-ng>

That’s it ! Key is broken. Now I will quickly go through WPA-PSK. Basically, the only way I found to attack it is a dictionary attack against the PSK.

The goal here is to capture the 4-way handshake. So do the de-auth as described above to cause the client to deauth and reconnect in hopes of catching the 4-way handshake. Sometimes this will take multiple tries to catch it. What I do is just keep on running aircrack against the active dump file to see if I got a handshake or not.
  
(You can also run ethereal on the file to see exactly what the handshake looks like just filter by EAPOL)

Once you got it. You can stop capturing traffic.

Now you can run aircrack with the WPA option and point it to your dictionary file. But I had troubles passing my very big dictionary file to it. So I then turned to cowpatty. Very straightforward, run it to see available options.

<http://sourceforge.net/projects/cowpatty>

Key found! Well I cheated a bit and put my PSK in the middle of the dictionary file.

This basically says that when using WPA-PSK, please people use a very good password. Something that will not be found in a normal dictionary file.

Well pretty simple huh? Almost to simple. Don&#8217;t need to much brains for this attack.

I am now starting to try to inject traffic with scapy. So if anyone has generated arp’s to wifi with it I would be interested to hear.

Refereneces:
  
<http://sourceforge.net/projects/cowpatty>
  
<http://www.aircrack-ng.org/doku.php>