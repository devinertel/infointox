---
id: 29
title: HTTP Verb Tampering
date: 2008-06-06T16:03:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=29
permalink: /?p=29
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2008/06/http-verb-tampering.html
categories:
  - HTTP Verb Tampering
  - Webscarab
tags:
  - Verb Tampering
---
While this is really nothing that new, it has recently resurfaced with some interesting uses. Aspect Security has published a white paper &#8220;[Bypassing Web Authentication and Authorization with HTTP Verb Tampering&#8221;](http://www.aspectsecurity.com/documents/Bypassing_VBAAC_with_HTTP_Verb_Tampering.pdf) that has brought this all to light.

In the past, I would usually use verb tampering for your XSS or SQLi attacks. Authorization bypass really never crossed my mind, this prompted me read up a bit. After some reading, It was kind of a &#8220;hit my head&#8221; type of moment. I highly recommend reading the white paper but here is a high level overview as I see it.

Verb Tampering for authorization bypass can be as simple as substituting a GET with a HEAD. For example:

In Java EE you can restrict access to a location with web.xml.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="xml" style="font-family:monospace;"><span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;security-constraint<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;web-resource-collection<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
      <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;url-pattern<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>/admin/*<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/url-pattern<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
      <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;http-method<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>GET<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/http-method<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
      <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;http-method<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>POST<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/http-method<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/web-resource-collection<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;auth-constraint<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
      <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;role-name<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>admin<span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/role-name<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
   <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/auth-constraint<span style="color: #000000; font-weight: bold;">&gt;</span></span></span>
 <span style="color: #009900;"><span style="color: #000000; font-weight: bold;">&lt;/security-constraint<span style="color: #000000; font-weight: bold;">&gt;</span></span></span></pre>
      </td>
    </tr>
  </table>
</div>

This restricts access to /admin and the listed http-methods. The thing that is not mentioned if you actually change the verb to something different such as HEAD, it will actually allow you access. You can easily do this with Webscarab by intercepting the request.

<a href="http://2.bp.blogspot.com/_CSYBzy3J1s8/SEmaN37bn0I/AAAAAAAAAbU/54GYWJ5TBY0/s1600-h/webscarab.JPG" onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}"><img id="BLOGGER_PHOTO_ID_5208864007172890434" style="cursor: pointer;" src="http://2.bp.blogspot.com/_CSYBzy3J1s8/SEmaN37bn0I/AAAAAAAAAbU/54GYWJ5TBY0/s320/webscarab.JPG" border="0" alt="" /></a>

Many people think by explicitly stating what methods to block that it won&#8217;t allow the others , when in fact if they did not state any methods it would block all. The methods that are listed are the only ones that are protected. Funny side note: I was actually looking at BEA docs for something totally unrelated and I happen to see their recommendation on how to secure folders with web.xml, and their way was vulnerable to the HEAD attack.

Another interesting find is the use of arbitrary verbs. In php and java this is allowed which means that we can throw it a verb that does not even exist. The application will then take the request and then convert it to a GET. This also bypasses the security restrictions.

More Reading:
  
<http://jeremiahgrossman.blogspot.com/2008/06/what-you-need-to-know-about-http-verb.html>
  
<http://www.webappsec.org/lists/websecurity/archive/2008-06/msg00022.html>
  
<http://www.webappsec.org/lists/websecurity/archive/2008-05/msg00072.html>
  
[http://www.aspectsecurity.com/documents/Bypassing\_VBAAC\_with\_HTTP\_Verb_Tampering.pdf](http://www.aspectsecurity.com/documents/Bypassing_VBAAC_with_HTTP_Verb_Tampering.pdf)
  
<http://www.securitycompass.com/exploit_me/accessme/accessme-0.1.shtml>
  
[http://www.aspectsecurity.com/documents/Aspect\_VBAAC\_Bypass.swf](http://www.aspectsecurity.com/documents/Aspect_VBAAC_Bypass.swf)