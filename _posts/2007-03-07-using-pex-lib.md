---
id: 24
title: Using PEX Lib
date: 2007-03-07T01:50:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=24
permalink: /?p=24
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2007/03/using-pex-lib_06.html
categories:
  - metasploit
  - PEX
tags:
  - metasploit
  - Perl
---
The metasploit framework has a handy little perl lib&nbsp; to aid you in finding your offset. There is nothing really fancy about it and many may already know about it.&nbsp; I figured for those who don&#8217;t, its useful enough to highlight. I did use this in my stack overflow example.&nbsp; 

At a very basic level lets say you got a seg fault by inserting 24 characters for input. With the Pex lib you can create a string 24 characters long all of unique dwords.

To create our pattern of&nbsp; 24 charcters we use pex like so.  
****  
perl -e &#8216;use Pex;print Pex::Text::PatternCreate(20)&#8217;****

or below is a&nbsp; little perl script that you can just&nbsp; run with&nbsp; 24 as the input. Ya I&#8217;m that lazy!

_#pex_pattern.pl  
#!/usr/bin/perl</p> 

#path to Pex lib in metasploit

use lib &#8220;/home/kpan1c/framework-2.6/lib/&#8221;;  
use Pex;

print Pex::Text::PatternCreate(@ARGV[0]). &#8220;\n&#8221;;  
</i>  
**./pex_pattern.pl&nbsp; 24**

Output will look like so  
_Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7</p> 

</i>Now you can just overflow the app with that string and then check EIP or whatever you wanted to overwrite. Once&nbsp; you see what is in it you can pass that string&nbsp; to patternoffset.pl located in the sdk dir of metasploit followed by the length of your pattern and bam you got an offset. Handy, and to think I use to create patterns like AAAABBBBCCCC&#8230;&#8230;  
****  
./patternOffset.pl 0x35614134 24****

References:  
<http://metasploit.com>  
<http://www.syngress.com/catalog/?pid=3270>