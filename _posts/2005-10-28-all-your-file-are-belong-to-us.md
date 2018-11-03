---
id: 7
title: All Your File Are Belong To Us
date: 2005-10-28T14:17:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=7
permalink: /?p=7
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2005/10/all-your-file-are-belong-to-us_28.html
categories:
  - Uncategorized
tags:
  - MITM
---
Been testing a neat little app called tcpxtract.
  
<http://tcpxtract.sourceforge.net/>

What it does is grab files from sniffed traffic though &#8220;carving&#8221;. Can be
  
used against live sniffing or against a pcap file.

Findings so far:

First, I thought I would run it against a kismet pcap file I had laying
  
around.
  
Turned up with a couple of images, must have been people browsing the web.
  
I would assume other files would work no problem, since wireless it is not a
  
switched network and all the traffic anyone can see.

1. FILES OWNED

Second, I thought I would fire up ethereal <http://www.ethereal.com/> and bind
  
it to my local Ethernet card to sniff.
  
I did a few file transfers during the sniff. SCP, FTP, Windows SMB Share(AD
  
Kerbros)
  
Saved the sniff in a pcap file and ran tcpxtract against it.

&#8211; SCP, I obviously did not grab that file I transferred.
  
&#8211; FTP, Do I even have to tell?
  
&#8211; SMB, Yep grabbed that file too

2. FILES OWNED

Third, I was thinking this isn&#8217;t that useful. Why do I want to see my own
  
files transferred and on
  
a wireless network anyone to transfer anything useful, is just plain stupid.

So, I got to thinking how about a &#8220;man in the middle&#8221; attack? I Fire up the
  
handy ettercap <http://ettercap.sourceforge.net/>
  
and poison the arp cache on the switch and route all traffic to my local
  
Ethernet card and then route the packets to their final destination.

Now since all the switch traffic is running though my Ethernet device. I
  
bind tcpxtract to my
  
local Ethernet device. And the files started to pour in (mpg, mp3,doc,pdf ,
  
etc) a lot.

3. FILES OWNED

Now, I&#8217;m sure people see the danger here. For security testers/auditors its
  
a way to rid your company of using
  
ftp and other non-secure protocols. Do that attack against some highly
  
sensitive servers, and then show your
  
manager all the nice sensitive documents you mined!

I will be looking into other methods of using tcpxract.