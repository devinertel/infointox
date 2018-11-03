---
id: 23
title: Nortel-Defaults.pl
date: 2006-11-08T21:57:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=23
permalink: /?p=23
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2006/11/nortel-defaultspl_08.html
categories:
  - Uncategorized
tags:
  - Perl
---
Well I wrote this a while ago for discovering default username and passwords on Nortel switches. It pretty much can be used for any telnet type device, although I think Cisco may need some more sleep()&#8217;s. I know its a dirty script, I needed it fast and figured why not post it. You can change the arrary of user/pass and/or have it go at different subnets and/or change where the switches are and/or scan every ip for a switch. In my case I knew the the octects of the switchs on every subnet.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="perl" style="font-family:monospace;"><span style="color: #666666; font-style: italic;">#!/usr/bin/perl</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">use</span>	Net<span style="color: #339933;">::</span><span style="color: #006600;">Telnet</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#Nortels default username/passwords</span>
<span style="color: #0000ff;">@norteldefault</span> <span style="color: #339933;">=</span> <span style="color: #009900;">&#40;</span><span style="color: #ff0000;">'rwa'</span><span style="color: #339933;">,</span><span style="color: #ff0000;">'rw'</span><span style="color: #339933;">,</span><span style="color: #ff0000;">'ro'</span><span style="color: #339933;">,</span><span style="color: #ff0000;">'l3'</span><span style="color: #339933;">,</span><span style="color: #ff0000;">'l2'</span><span style="color: #339933;">,</span><span style="color: #ff0000;">'l1'</span><span style="color: #339933;">,</span><span style="color: #ff0000;">'operator'</span><span style="color: #339933;">,</span><span style="color: #ff0000;">'slbop'</span><span style="color: #339933;">,</span><span style="color: #ff0000;">'slbadmin'</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#all switches ip</span>
<span style="color: #0000ff;">$top</span><span style="color: #339933;">=</span><span style="color: #cc66cc;">254</span><span style="color: #339933;">;</span>
<span style="color: #0000ff;">$bot</span><span style="color: #339933;">=</span><span style="color: #cc66cc;">126</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#gernerate hosts to test</span>
<span style="color: #b1b100;">for</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">$a</span><span style="color: #339933;">=</span> <span style="color: #cc66cc;">1</span><span style="color: #339933;">;</span> <span style="color: #0000ff;">$a</span> <span style="color: #339933;">&</span><span style="color: #b1b100;">lt</span><span style="color: #339933;">;</span> <span style="color: #cc66cc;">100</span><span style="color: #339933;">;</span> <span style="color: #0000ff;">$a</span><span style="color: #339933;">++</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#123;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#creat host for every class we want to scan and push on array</span>
<span style="color: #666666; font-style: italic;">#just comment out blocks you dont want to scan or add more</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#192.168.*.1,253</span>
<span style="color: #666666; font-style: italic;">#$temp ="192.1.".$a.".".$top;</span>
<span style="color: #666666; font-style: italic;">#push @hosts,$temp;</span>
<span style="color: #666666; font-style: italic;">#$temp ="192.168.".$a.".".$bot;</span>
<span style="color: #666666; font-style: italic;">#push @hosts,$temp;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#192.1.*.1,253</span>
<span style="color: #666666; font-style: italic;">#$temp ="192.1.".$a.".".$top;</span>
<span style="color: #666666; font-style: italic;">#push @hosts,$temp;</span>
<span style="color: #666666; font-style: italic;">#$temp ="192.1.".$a.".".$bot;</span>
<span style="color: #666666; font-style: italic;">#push @hosts,$temp;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#10.10.*.1,253</span>
<span style="color: #0000ff;">$temp</span> <span style="color: #339933;">=</span><span style="color: #ff0000;">"10.10."</span><span style="color: #339933;">.</span><span style="color: #0000ff;">$a</span><span style="color: #339933;">.</span><span style="color: #ff0000;">"."</span><span style="color: #339933;">.</span><span style="color: #0000ff;">$top</span><span style="color: #339933;">;</span>
<span style="color: #000066;">push</span> <span style="color: #0000ff;">@hosts</span><span style="color: #339933;">,</span><span style="color: #0000ff;">$temp</span><span style="color: #339933;">;</span>
<span style="color: #0000ff;">$temp</span> <span style="color: #339933;">=</span><span style="color: #ff0000;">"10.10."</span><span style="color: #339933;">.</span><span style="color: #0000ff;">$a</span><span style="color: #339933;">.</span><span style="color: #ff0000;">"."</span><span style="color: #339933;">.</span><span style="color: #0000ff;">$bot</span><span style="color: #339933;">;</span>
<span style="color: #000066;">push</span> <span style="color: #0000ff;">@hosts</span><span style="color: #339933;">,</span><span style="color: #0000ff;">$temp</span><span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#setup telnet</span>
<span style="color: #0000ff;">$telnet</span> <span style="color: #339933;">=</span> <span style="color: #000000; font-weight: bold;">new</span> Net<span style="color: #339933;">::</span><span style="color: #006600;">Telnet</span> <span style="color: #009900;">&#40;</span>Timeout <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #cc66cc;">3</span><span style="color: #339933;">,</span> Errmode <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #ff0000;">"return"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#main loop to do the fun</span>
<span style="color: #b1b100;">foreach</span> <span style="color: #0000ff;">$host</span> <span style="color: #009900;">&#40;</span><span style="color: #0000ff;">@hosts</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#123;</span>
	<span style="color: #000066;">chomp</span> <span style="color: #0000ff;">$host</span><span style="color: #339933;">;</span>
	<span style="color: #b1b100;">if</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">$telnet</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #000066;">open</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">$host</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#123;</span>
		<span style="color: #000066;">print</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>Connected to $host"</span><span style="color: #339933;">;</span>
&nbsp;
		<span style="color: #b1b100;">foreach</span> <span style="color: #0000ff;">$userpass</span> <span style="color: #009900;">&#40;</span><span style="color: #0000ff;">@norteldefault</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
			<span style="color: #000066;">chomp</span> <span style="color: #0000ff;">$userpass</span><span style="color: #339933;">;</span>
			<span style="color: #0000ff;">$user</span> <span style="color: #339933;">=</span> <span style="color: #0000ff;">$userpass</span><span style="color: #339933;">;</span>
			<span style="color: #0000ff;">$pass</span> <span style="color: #339933;">=</span> <span style="color: #0000ff;">%userpass</span><span style="color: #339933;">;</span>
			<span style="color: #0000ff;">&amp</span><span style="color: #339933;">;</span>login<span style="color: #339933;">;</span>
			<span style="color: #000066;">sleep</span> <span style="color: #009900;">&#40;</span><span style="color: #cc66cc;">30</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>	
		<span style="color: #009900;">&#125;</span><span style="color: #339933;">;</span>
	<span style="color: #009900;">&#125;</span>
	<span style="color: #b1b100;">else</span><span style="color: #009900;">&#123;</span><span style="color: #000066;">print</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>Could Not Connect To $host"</span><span style="color: #009900;">&#125;</span>
<span style="color: #009900;">&#125;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">####old testing </span>
<span style="color: #666666; font-style: italic;">#sub conn{</span>
<span style="color: #666666; font-style: italic;">#</span>
<span style="color: #666666; font-style: italic;">#$telnet = new Net::Telnet (Timeout =&gt; 3, Errmode =&gt; "return");</span>
<span style="color: #666666; font-style: italic;">#if($telnet-&gt; open($host)){</span>
<span style="color: #666666; font-style: italic;">#	$connect=1;</span>
<span style="color: #666666; font-style: italic;">#	&amp;login;</span>
<span style="color: #666666; font-style: italic;">#}</span>
<span style="color: #666666; font-style: italic;">#};</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">sub</span> login<span style="color: #009900;">&#123;</span>
<span style="color: #000066;">print</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>Trying To login with $userpass"</span><span style="color: #339933;">;</span>
<span style="color: #000066;">print</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>Waiting 30sec before next guess. prevent susp. and lockouts"</span><span style="color: #339933;">;</span>
<span style="color: #b1b100;">if</span><span style="color: #009900;">&#40;</span><span style="color: #0000ff;">$telnet</span> <span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> login<span style="color: #009900;">&#40;</span><span style="color: #0000ff;">$userpass</span><span style="color: #339933;">,</span><span style="color: #0000ff;">$userpass</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#123;</span>
	<span style="color: #000066;">print</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>Logged In With $userpass To $host !!!!"</span><span style="color: #339933;">;</span>
	<span style="color: #000066;">print</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>This has been logged to File!!!!"</span><span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span>
<span style="color: #0000ff;">$telnet</span> <span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #000066;">close</span><span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span><span style="color: #339933;">;</span></pre>
      </td>
    </tr>
  </table>
</div>

**
  
Reference:**
  
<http://search.cpan.org/~jrogers/Net-Telnet-3.03/lib/Net/Telnet.pm>