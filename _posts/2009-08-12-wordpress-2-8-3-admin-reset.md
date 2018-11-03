---
id: 34
title: 'WordPress &lt;= 2.8.3 Admin Reset'
date: 2009-08-12T22:17:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=34
permalink: /?p=34
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2009/08/wordpress-283-admin-reset.html
categories:
  - Uncategorized
tags:
  - SQLi
---
<a href="http://2.bp.blogspot.com/_CSYBzy3J1s8/SoNA32tFXjI/AAAAAAAAAtg/Y-c03xOEgrI/s1600-h/wordpress-logo.png" onblur="try {parent.deselectBloggerImageGracefully();} catch(e) {}"><img id="BLOGGER_PHOTO_ID_5369206509077356082" style="float: right; margin: 0pt 0pt 10px 10px; cursor: pointer; width: 200px; height: 188px;" src="http://2.bp.blogspot.com/_CSYBzy3J1s8/SoNA32tFXjI/AAAAAAAAAtg/Y-c03xOEgrI/s200/wordpress-logo.png" border="0" alt="" /></a>None of this is new its just me trying to understand it. This vulnerability only resets the admin password, which is then emailed to the admin. Someone could potentially DOS the admin with a small script to continually reset the password but overall this is just an annoyance. This is mainly due to a lack of input validation on the $key variable. How this seems to work is WordPress is using a black list method to check to see if the key is empty and it also has no checks to see if the key is empty before the query is ran.

<span style="font-weight: bold;">Proof of Concept:</span>
  
http://DOMAINNAME/wp-login.php?action=rp&key[]=

<span style="font-weight: bold;">Why does it reset admin? </span>

When $key is passed an array[] it is treated an empty string. This will in turn match every user within the database. The first user just happens to be the admin, which WordPress will reset.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="php" style="font-family:monospace;"><span style="color: #000088;">$user</span> <span style="color: #339933;">=</span> <span style="color: #000088;">$wpdb</span><span style="color: #339933;">-&</span>gt<span style="color: #339933;">;</span>get_row<span style="color: #009900;">&#40;</span>
<span style="color: #000088;">$wpdb</span><span style="color: #339933;">-&</span>gt<span style="color: #339933;">;</span>prepare<span style="color: #009900;">&#40;</span>
<span style="color: #0000ff;">"SELECT * FROM <span style="color: #006699; font-weight: bold;">$wpdb</span>-&gt;users
WHERE user_activation_key = <span style="color: #009933; font-weight: bold;">%s</span>"</span><span style="color: #339933;">,</span> <span style="color: #000088;">$key</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span></pre>
      </td>
    </tr>
  </table>
</div>

<span style="font-weight: bold;">The Issue.</span>
  
It looks like empty() will treat an array as an empty string and not return an error.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="php" style="font-family:monospace;"><span style="color: #666666; font-style: italic;">#wp-login.php.
</span><span style="color: #b1b100;">if</span> <span style="color: #009900;">&#40;</span> <span style="color: #990000;">empty</span><span style="color: #009900;">&#40;</span> <span style="color: #000088;">$key</span> <span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#41;</span>
<span style="color: #b1b100;">return</span> <span style="color: #000000; font-weight: bold;">new</span> WP_Error<span style="color: #009900;">&#40;</span><span style="color: #0000ff;">'invalid_key'</span><span style="color: #339933;">,</span> __<span style="color: #009900;">&#40;</span><span style="color: #0000ff;">'Invalid key'</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span></pre>
      </td>
    </tr>
  </table>
</div>

<span style="font-weight: bold;">The Fix.</span>
  
WordPress has released a [fix](http://wordpress.org/development/2009/08/2-8-4-security-release/) which is shown below. This is still a black list approach and only adds an extra check for the array.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="php" style="font-family:monospace;"><span style="color: #b1b100;">if</span> <span style="color: #009900;">&#40;</span> <span style="color: #990000;">empty</span><span style="color: #009900;">&#40;</span> <span style="color: #000088;">$key</span> <span style="color: #009900;">&#41;</span> <span style="color: #339933;">||</span> <span style="color: #990000;">is_array</span><span style="color: #009900;">&#40;</span> <span style="color: #000088;">$key</span> <span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#41;</span>
<span style="color: #b1b100;">return</span> <span style="color: #000000; font-weight: bold;">new</span> WP_Error<span style="color: #009900;">&#40;</span><span style="color: #0000ff;">'invalid_key'</span><span style="color: #339933;">,</span> __<span style="color: #009900;">&#40;</span><span style="color: #0000ff;">'Invalid key'</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span></pre>
      </td>
    </tr>
  </table>
</div>

This is still using a black list method and I also think some improvements can be made before the query statement. I believe some blame can be put on PHP by not throwing an exception to an empty array. When time permits I would like to play around with other things that could be passed to $key. I&#8217;m still exploring other possibilities of this not just being a password reset that is sent to the admin. If anyone has some ideas, I would love to hear.

<span style="font-weight: bold;"><br /> References:</span>
  
[
  
http://core.trac.wordpress.org/changeset/11798](http://core.trac.wordpress.org/changeset/11798)
  
<http://archives.neohapsis.com/archives/fulldisclosure/2009-08/0114.html>
  
<http://us3.php.net/manual/en/function.empty.php>
  
<http://isc.sans.org/diary.html?storyid=6934><!--more-->

<!--more-->

<!--more-->