---
id: 15
title: webserv-naslgrab.nasl
date: 2006-01-11T17:22:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=15
permalink: /?p=15
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2006/01/webserv-naslgrabnasl_11.html
categories:
  - Uncategorized
tags:
  - NASL
---
Well I wanted to get a bit more familiar with NASL (Nessus Attack Scripting Language). I&#8217;ve modified nessus plugins in the past but never really did much with it. I have to say I do like it, pretty easy to do testing with.

I needed a way to check a lot of webservers for their versions, and fast. So figured what the heck let me throw something together with NASL. Now this is just a stand-alone script, it will not work within the nessus framework.
  
(More docs to work with nessus framework are below)

This just sends a HEAD request to the webserver and greps for the server string.

This also could be easly modified to read from the socket and grab other banners. I found this would work for telnet, ftp ,ssh, etc. but for some reason I could not grab the banner from the webservers I was testing. Hence sending
  
&#8220;HEAD / HTTP/1.0\r\n\r\n&#8221;

If you wanted to read right from the socket without sending the HEAD command you could just comment that out and replace name w/ server.

I will be looking into this more, but this was just a quick script to get my feet wet.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="perl" style="font-family:monospace;">&nbsp;
<span style="color: #666666; font-style: italic;">#####################################################################</span>
<span style="color: #666666; font-style: italic;"># Name: webserv-naslgrab.nasl					    #</span>
<span style="color: #666666; font-style: italic;"># Description: A non-intrusive way to grab the web server version   #</span>
<span style="color: #666666; font-style: italic;">#              by sending opening a socket to 80 and sending a      #</span>
<span style="color: #666666; font-style: italic;">#	       HEAD Request. This can be modified to use other      #</span>
<span style="color: #666666; font-style: italic;">#	       ports.                                               #</span>
<span style="color: #666666; font-style: italic;"># Version: .1                                                       #</span>
<span style="color: #666666; font-style: italic;"># Author : Devin Ertel                                              #</span>
<span style="color: #666666; font-style: italic;"># Usage	 : nasl -t 192.168.1-155 webserv-naslgrab.nasl              #</span>
<span style="color: #666666; font-style: italic;">#####################################################################</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#Create tcp socket to port 80</span>
soc <span style="color: #339933;">=</span> open_sock_tcp<span style="color: #009900;">&#40;</span><span style="color: #cc66cc;">80</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#grab host ip of current box with socket open</span>
hostip<span style="color: #339933;">=</span>get_host_ip<span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#if socket was created</span>
<span style="color: #b1b100;">if</span> <span style="color: #009900;">&#40;</span>soc<span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#create string and send</span>
str <span style="color: #339933;">=</span> string<span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"HEAD / HTTP/1.0<span style="color: #000099; font-weight: bold;">\r</span><span style="color: #000099; font-weight: bold;">\n</span><span style="color: #000099; font-weight: bold;">\r</span><span style="color: #000099; font-weight: bold;">\n</span>"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #000066;">send</span><span style="color: #009900;">&#40;</span><span style="color: #000066;">socket</span><span style="color: #339933;">:</span>soc<span style="color: #339933;">,</span> data<span style="color: #339933;">:</span>str<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#grab data from the socket</span>
name <span style="color: #339933;">=</span> <span style="color: #000066;">recv</span><span style="color: #009900;">&#40;</span><span style="color: #000066;">socket</span><span style="color: #339933;">:</span>soc<span style="color: #339933;">,</span> <span style="color: #000066;">length</span><span style="color: #339933;">:</span><span style="color: #cc66cc;">1024</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#grep for the line with server in it</span>
server <span style="color: #339933;">=</span> egrep<span style="color: #009900;">&#40;</span>pattern<span style="color: #339933;">:</span><span style="color: #ff0000;">"Server.*"</span><span style="color: #339933;">,</span> string <span style="color: #339933;">:</span> name<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#if grep returns value</span>
<span style="color: #b1b100;">if</span><span style="color: #009900;">&#40;</span>server<span style="color: #009900;">&#41;</span><span style="color: #009900;">&#123;</span>
display<span style="color: #009900;">&#40;</span>server<span style="color: #339933;">,</span><span style="color: #ff0000;">" On IP "</span><span style="color: #339933;">,</span>hostip<span style="color: #339933;">,</span><span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#close socket</span>
<span style="color: #000066;">close</span><span style="color: #009900;">&#40;</span>soc<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span></pre>
      </td>
    </tr>
  </table>
</div>

**Links:**
  
<http://michel.arboi.free.fr/nasl2ref/>
  
<http://www.oreillynet.com/pub/a/security/2004/06/03/nessus_plugins.html>
  
<http://www.virtualblueness.net/nasl.html>