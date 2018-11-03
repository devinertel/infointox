---
id: 31
title: WAF Checker
date: 2008-08-08T23:30:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=31
permalink: /?p=31
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2008/08/waf-checker.html
categories:
  - NASL
  - WAF
tags:
  - NASL
  - WAF
---
During a large application assessment, I noticed in a cookie that it was load balanced. I gathered as many unique cookies I could and noticed the application was spread across many web servers. This allows room for errors concerning a WAF. Why not attack a server that the WAF is not protecting?

On this note I wrote a quick little NASL script to find a server that is not protected by the WAF. The only trick to this script is understanding what response you get once the WAF is triggered. Every WAF I have worked with all block the User Agent &#8220;nikto&#8221; by default. To find the response it gives I just set my User Agent to &#8220;nikto&#8221; and make a standard GET request. If this doesn&#8217;t work you can call your basic XSS stuff and it will usually trigger the WAF.

Once you can get the WAF triggered you just have to find something different in the response for the script to look for. In my case it was just &#8220;error&#8221;. Now just run the script and see if any WAF&#8217;s are down.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="ruby" style="font-family:monospace;"><span style="color:#008000; font-style:italic;">#Create tcp socket to webserver port</span>
socket_timeout = <span style="color:#006666;">5</span>;
soc = open_sock_tcp<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#006666;">80</span><span style="color:#006600; font-weight:bold;">&#41;</span>;
&nbsp;
<span style="color:#008000; font-style:italic;">#grab host ip of current box with socket open</span>
hostip=get_host_ip<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#006600; font-weight:bold;">&#41;</span>;
&nbsp;
<span style="color:#008000; font-style:italic;">#if socket was created</span>
<span style="color:#9966CC; font-weight:bold;">if</span> <span style="color:#006600; font-weight:bold;">&#40;</span>soc<span style="color:#006600; font-weight:bold;">&#41;</span> <span style="color:#006600; font-weight:bold;">&#123;</span>
&nbsp;
<span style="color:#008000; font-style:italic;">#create string and send</span>
str = <span style="color:#CC0066; font-weight:bold;">string</span><span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#996600;">"GET /index.html HTTP/1.0<span style="color:#000099;">\r</span><span style="color:#000099;">\n</span>User-Agent:Nikto<span style="color:#000099;">\r</span><span style="color:#000099;">\n</span><span style="color:#000099;">\r</span><span style="color:#000099;">\n</span>"</span><span style="color:#006600; font-weight:bold;">&#41;</span>;
send<span style="color:#006600; font-weight:bold;">&#40;</span>socket:soc, data:str<span style="color:#006600; font-weight:bold;">&#41;</span>;
&nbsp;
<span style="color:#008000; font-style:italic;">#grab data from the socket</span>
page = recv<span style="color:#006600; font-weight:bold;">&#40;</span>socket:soc, length:<span style="color:#006666;">4096</span><span style="color:#006600; font-weight:bold;">&#41;</span>;
&nbsp;
<span style="color:#008000; font-style:italic;">#grep for the line with error or whatever waf refturns</span>
error = egrep<span style="color:#006600; font-weight:bold;">&#40;</span>pattern:<span style="color:#996600;">"error*"</span>, <span style="color:#CC0066; font-weight:bold;">string</span> : page<span style="color:#006600; font-weight:bold;">&#41;</span>;
&nbsp;
<span style="color:#008000; font-style:italic;">#if grep returns value</span>
<span style="color:#9966CC; font-weight:bold;">if</span><span style="color:#006600; font-weight:bold;">&#40;</span>error<span style="color:#006600; font-weight:bold;">&#41;</span><span style="color:#006600; font-weight:bold;">&#123;</span>
display<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#996600;">"WAF ON "</span>,hostip,<span style="color:#996600;">"<span style="color:#000099;">\n</span>"</span><span style="color:#006600; font-weight:bold;">&#41;</span>;
<span style="color:#006600; font-weight:bold;">&#125;</span>
<span style="color:#9966CC; font-weight:bold;">else</span><span style="color:#006600; font-weight:bold;">&#123;</span>
display<span style="color:#006600; font-weight:bold;">&#40;</span><span style="color:#996600;">"WAF OFF "</span>,hostip,<span style="color:#996600;">"<span style="color:#000099;">\n</span>"</span><span style="color:#006600; font-weight:bold;">&#41;</span>;
<span style="color:#006600; font-weight:bold;">&#125;</span>
&nbsp;
<span style="color:#008000; font-style:italic;">#close socket</span>
close<span style="color:#006600; font-weight:bold;">&#40;</span>soc<span style="color:#006600; font-weight:bold;">&#41;</span>;
<span style="color:#006600; font-weight:bold;">&#125;</span></pre>
      </td>
    </tr>
  </table>
</div>

I found this script pretty handy for pen-testing and monitoring.

On the monitoring side you can just throw it in a cron job and have it email you if any WAF&#8217;s were found to be off.

On the pen-testing side its a lot easier attacking an app with out those pesky WAF&#8217;s