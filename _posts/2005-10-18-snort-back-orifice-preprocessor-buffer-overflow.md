---
id: 6
title: Snort Back Orifice Preprocessor Buffer Overflow
date: 2005-10-18T14:47:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=6
permalink: /?p=6
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2005/10/snort-back-orifice-preprocessor-buffer_18.html
categories:
  - Uncategorized
tags:
  - Buffer Overflow
  - Snort
---
While looking into US-CERT TA-05-291A. This is what I found.

While snort does review the traffic on port 31337, it will also look
  
for any UDP traffic that is using Back Orifice&#8217;s magic cookie.

* spp_bo.c comments
  
*
  
* Purpose: Detects Back Orifice traffic by brute forcing the weak encryption
  
* of the program&#8217;s network protocol and detects the magic cookie
  
* that it&#8217;s servers and clients require to communicate with each
  
* other.
  
*
  
\* Back Orifice magic cookie is &#8220;\*!*QWTY?&#8221;, which is located in the first
  
* eight bytes of the packet. But it is encrypted using an XOR.

When exploiting this we want this function of the preprocessor to kick
  
off. Which is why you will have to create a UDP packet that is not
  
using port 31337.

Below is where the fun happens.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="c" style="font-family:monospace;"><span style="color: #666666; font-style: italic;">//snippet from spp_bo.c</span>
<span style="color: #993333;">static</span> <span style="color: #993333;">int</span> BoGetDirection<span style="color: #009900;">&#40;</span>Packet <span style="color: #339933;">*</span>p<span style="color: #339933;">,</span> <span style="color: #993333;">char</span> <span style="color: #339933;">*</span>pkt_data<span style="color: #009900;">&#41;</span>
<span style="color: #009900;">&#123;</span>
u_int32_t len <span style="color: #339933;">=</span> <span style="color: #0000dd;"></span><span style="color: #339933;">;</span>
u_int32_t id <span style="color: #339933;">=</span> <span style="color: #0000dd;"></span><span style="color: #339933;">;</span>
u_int32_t l<span style="color: #339933;">,</span> i<span style="color: #339933;">;</span>
<span style="color: #993333;">char</span> type<span style="color: #339933;">;</span>
<span style="color: #993333;">char</span> buf1<span style="color: #009900;">&#91;</span><span style="color: #0000dd;">1024</span><span style="color: #009900;">&#93;</span><span style="color: #339933;">;</span> <span style="color: #339933;">#Interesting ??? A static array? Is this checked? hehe</span>
<span style="color: #993333;">char</span> buf2<span style="color: #009900;">&#91;</span><span style="color: #0000dd;">1024</span><span style="color: #009900;">&#93;</span><span style="color: #339933;">;</span> <span style="color: #339933;">#Interesting ??? A static array? Is this checked? hehe</span>
<span style="color: #993333;">char</span> <span style="color: #339933;">*</span>buf_ptr<span style="color: #339933;">;</span>
<span style="color: #993333;">char</span> plaintext<span style="color: #339933;">;</span>
<span style="color: #666666; font-style: italic;">//snippet from spp_bo.c</span>
&nbsp;
I don<span style="color: #ff0000;">'t see any checks.
&nbsp;
//snippet from spp_bo.c
/* Only examine data if this a ping request or response */
if ( type == BO_TYPE_PING )
{
i = 0;
buf_ptr = buf1;
*buf1 = 0;
*buf2 = 0;
/* Decrypt data */
while ( i &lt; len )
{
plaintext = (char) (*pkt_data ^ (BoRand()%256));
*buf_ptr = plaintext;
i++;
pkt_data++;
buf_ptr++;
if ( plaintext == 0 )
buf_ptr = buf2;
}
&nbsp;
/* null-terminate string */
*buf_ptr = 0;
&nbsp;
DEBUG_WRAP(DebugMessage(DEBUG_PLUGIN, "buf1 = %s<span style="color: #000099; font-weight: bold;">\n</span>", buf1););
&nbsp;
if ( *buf2 != 0 )
{
DEBUG_WRAP(DebugMessage(DEBUG_PLUGIN, "buf2 = %s<span style="color: #000099; font-weight: bold;">\n</span>",buf2););
}
&nbsp;
DEBUG_WRAP(DebugMessage(DEBUG_PLUGIN, "crc = 0x%x<span style="color: #000099; font-weight: bold;">\n</span>", (char)
(*pkt_data ^ (BoRand()%256))););
&nbsp;
if ( len &gt; 4 &amp;&amp; !strncasecmp((buf1+3), "PONG", 4) )
{
return BO_FROM_SERVER;
}
else
{
return BO_FROM_CLIENT;
}
}
//snippet from spp_bo.c</span></pre>
      </td>
    </tr>
  </table>
</div>

To validate it a bit more I ran the code through flawfinder. This is
  
the output.

Examining spp_bo.c
  
spp_bo.c:430: \[2\] (buffer) char:
  
Statically-sized arrays can be overflowed. Perform bounds checking,
  
use functions that limit length, or ensure that the size is larger
  
than
  
the maximum possible length.
  
spp_bo.c:431: \[2\] (buffer) char:
  
Statically-sized arrays can be overflowed. Perform bounds checking,
  
use functions that limit length, or ensure that the size is larger
  
than
  
the maximum possible length.

Just my findings.