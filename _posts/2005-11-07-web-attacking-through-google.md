---
id: 10
title: Web Attacking Through Google
date: 2005-11-07T21:55:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=10
permalink: /?p=10
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2005/11/web-attacking-through-google_07.html
categories:
  - Uncategorized
tags:
  - zero packet
---
An attacker just may be able to do web based attacks through google. The goal of the attacker would be to have google process the malicious request against the target.

I first tested it with the ad-content section of the personal google page, which seems it does at least need some type of RSS content to process it.

For example, Here is a very very basic directory traversal attack:
  
<http://target/showfile.pl?f=../../../fileyouwant>

At first I was thinking you can add this to &#8220;add content&#8221; on the personal page. Didn&#8217;t seem to work. Like I said earlier it does want some type of RSS content.

So I then tried something like this in &#8220;add content&#8221;.
  
<http://rsssite/rss.php?xml+http://target/showfile.pl?f=../../../fileyouwant>

Still no go.

So then I thought google caching.
  
Basically you setup a static html page on some free web hosting company,the page would have all of the attack links(directory traversals,sql injections,php exploits, etc.)

Wait for google to cache it.
  
Viewing the page through google cache, the attacker could then launch all of the attacks from google.
  
This would all be done with a point and click.

Not really that dangerous since if you really wanted to find the attacker, google could provide you with logs. But it would be from an anonymous web site and would be two more steps.(could even proxy the registration of the site)

It would look weird for the person watching the IDS(google attacking??) some may even not think anything of it,thinking its the just the googlebot. Also, it would be hard for the target to block you considering most places do not want to block google.

This just kind of follows up Johnny Long&#8217;s idea of zero-packet attacks.
  
<http://www.blackhat.com/presentations/bh-usa-05/bh-us-05-long.pdf>