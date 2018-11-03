---
id: 14
title: GPG Signature Checking w/ Debian And Apt 0.6
date: 2006-01-07T10:16:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=14
permalink: /?p=14
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2006/01/gpg-signature-checking-w-debian-and-apt_07.html
categories:
  - Uncategorized
tags:
  - Linux
---
In new versions of apt. GPG signature checking is enabled by default. This is a good thing, allowing us trust the packages we are installing on our system. But if you recently updgraded apt it will begin to complain about not being able to find the pubkey. It should look something like this.

**W: GPG error: http://mirrors.kernel.org testing Release: The following signatures couldn&#8217;t be verified because the public key is not available: NO_PUBKEY 010908312D230C5F**

Now I just recently got sick of seeing this message and decided I should really get this working for security reasons. That being said I am a bit new to this. There are many documents out there on how to get this working, I am just documenting different things I did and found. Hopefully once I fully understand the process I will clean this up.

First, you will need to have a 0.6 version of apt and gnupgp installed.

Once you have that an easy way to try and fix this problem is install the debian-keyring.  
**  
apt-get install debian-keyring**

Now we can import the key using apt-key.

**  
apt-key add /usr/share/keyrings/debian-keyring.gpg  
apt-key add /usr/share/keyrings/debian-role-keys.gpg  
** 

Now I am really not sure what the difference is between them.  
This fixed my message of NO_PUBKEY for ftp.nerim.net  
If you don&#8217;t have this in your sources.list and its a desktop. I really would add this, alot of video and media type debs that debian does not carry. Link below:  
**http://debian.video.free.fr**

Ok, back to buisness. apt is still complaining about my kernel.org mirror.  
I read somewhere that a new key will added every year on the year. Somehow I did not have that new key from the debian-keyring package. So lets go get it.

**  
wget http://ftp-master.debian.org/ziyi\_key\__year_.asc  
** 

Now I just added it with apt-key but you could just do it with gpg.

**  
gpg &#8211;import ziyi\_key\_2006.asc</p> 

or

apt-key add ziyi\_key\_2006.asc  
</b>

Now apt-get update and it should be fixed. Like I said earlier, I&#8217;m a little unclear about this whole process. I would have thought downloading the whole debian-keyring would have done it. It is even over kill becuase I really only needed a couple of keys.

Links:  
<http://www.debian-administration.org/articles/174>  
<http://secure-testing-master.debian.net/>  
<http://lists.debian.org/debian-user/2005/11/msg00064.html>  
<http://moonbase.rydia.net/mental/blog/life/mixing-ubuntu-and-debian.html>