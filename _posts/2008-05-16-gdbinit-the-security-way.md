---
id: 28
title: .gdbinit The Security Way
date: 2008-05-16T15:15:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=28
permalink: /?p=28
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2008/05/gdbinit-security-way.html
categories:
  - debug
  - gdb
  - gdbinit
tags:
  - gdb
---
`I have worked quite a bit with gdb and have become pretty familiar with the common commands. gdb is a great debugger for * nix systems, but it can become a bit overwhelming with all the different commands. To make this a bit easier and more security friendly we can use <span style="font-style: italic;">~.gdbinit</span>. Within <span style="font-style: italic;">.gdbinit</span> you can define alias and macros to aid in displaying and use of gdb.`

With a good gdbinit file, life will be a bit more pleasent when searching for that next vulnerbility and/or reversing that next file.

Lets start with creating an alias.

<span style="font-weight: bold;">define bpl</span>
  
 <span style="font-weight: bold;">info breakpoints</span>
  
<span style="font-weight: bold;">end</span>

Here is another one that save your fingers from typing info all the time.

<span style="font-weight: bold;">define stack</span>
  
 <span style="font-weight: bold;">info stack</span>
  
 <span style="font-weight: bold;">info frame</span>
  
 <span style="font-weight: bold;">info args</span>
  
 <span style="font-weight: bold;">info locals</span>
  
<span style="font-weight: bold;">end</span>

 <span style="font-weight: bold;"></span>None of this is anything new, but in all my readings I have just recently learned about it. So I figured posting something may help out. I always knew there was a way to create macros in gdb, it just never dawned on me to look in <span style="font-style: italic;">.gdbinit </span>!

To make life simple and if you don&#8217;t want to write your own, below is a nice cracker-friendly <span style="font-style: italic;">.gdbinit</span> file I found.

<div style="height: 350px; white-space: pre-wrap; overflow: auto; padding: 8px;">
  <div class="wp_syntax">
    <table>
      <tr>
        <td class="code">
          <pre class="bash" style="font-family:monospace;"><span style="color: #666666; font-style: italic;"># INSTRUCTIONS: save as ~/.gdbinit</span>
<span style="color: #666666; font-style: italic;"># ______________breakpoint aliases_____________</span>
define bpl
info breakpoints
end
document bpl
List breakpoints
end
&nbsp;
define bp
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_CONTEXT</span> = <span style="color: #000000;">1</span>
<span style="color: #7a0874; font-weight: bold;">break</span> <span style="color: #000000; font-weight: bold;">*</span> <span style="color: #007800;">$arg0</span>
end
document bp
Set a breakpoint on address
Usage: bp addr
end
&nbsp;
define bpc
<span style="color: #c20cb9; font-weight: bold;">clear</span> <span style="color: #007800;">$arg0</span>
end
document bpc
Clear breakpoint at function<span style="color: #000000; font-weight: bold;">/</span>address
Usage: bpc addr
end
&nbsp;
define bpe
<span style="color: #7a0874; font-weight: bold;">enable</span> <span style="color: #007800;">$arg0</span>
end
document bpe
Enable breakpoint <span style="color: #666666; font-style: italic;">#</span>
Usage: bpe num
end
&nbsp;
define bpd
disable <span style="color: #007800;">$arg0</span>
end
document bpd
Disable breakpoint <span style="color: #666666; font-style: italic;">#</span>
Usage: bpd num
end
&nbsp;
define bpt
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_CONTEXT</span> = <span style="color: #000000;">1</span>
tbreak <span style="color: #007800;">$arg0</span>
end
document bpt
Set a temporary breakpoint on address
Usage: bpt addr
end
&nbsp;
define bpm
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_CONTEXT</span> = <span style="color: #000000;">1</span>
awatch <span style="color: #007800;">$arg0</span>
end
document bpm
Set a read<span style="color: #000000; font-weight: bold;">/</span><span style="color: #c20cb9; font-weight: bold;">write</span> breakpoint on address
Usage: bpm addr
end
&nbsp;
define bhb
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_CONTEXT</span> = <span style="color: #000000;">1</span>
hb <span style="color: #000000; font-weight: bold;">*</span> <span style="color: #007800;">$arg0</span>
end
document bhb
Set Hardware Assisted breakpoint on address
Usage: bhb addr
end
&nbsp;
<span style="color: #666666; font-style: italic;"># ______________process information____________</span>
define argv
show args
end
document argv
Print program arguments
end
&nbsp;
define stack
info stack
end
document stack
Print call stack
end
&nbsp;
define frame
info frame
info args
info locals
end
document frame
Print stack frame
end
&nbsp;
define flags
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0xB<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"O "</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"o "</span>
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0xA<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"D "</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"d "</span>
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">9</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"I "</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"i "</span>
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">8</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"T "</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"t "</span>
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">7</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"S "</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"s "</span>
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">6</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Z "</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"z "</span>
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">4</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"A "</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"a "</span>
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">2</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"P "</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"p "</span>
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"C "</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"c "</span>
end
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>"</span>
end
document flags
Print flags register
end
&nbsp;
define eflags
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" OF &lt;%d&gt; DF &lt;%d&gt; IF &lt;%d&gt; TF &lt;%d&gt;"</span>,\
<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0xB<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0xA<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, \
<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">9</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">8</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" SF &lt;%d&gt; ZF &lt;%d&gt; AF &lt;%d&gt; PF &lt;%d&gt; CF &lt;%d&gt;<span style="color: #000099; font-weight: bold;">\n</span>"</span>,\
<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">7</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">6</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>,\
<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">4</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">2</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" ID &lt;%d&gt; VIP &lt;%d&gt; VIF &lt;%d&gt; AC &lt;%d&gt;"</span>,\
<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0x15<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0x14<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, \
<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0x13<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0x12<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" VM &lt;%d&gt; RF &lt;%d&gt; NT &lt;%d&gt; IOPL &lt;%d&gt;<span style="color: #000099; font-weight: bold;">\n</span>"</span>,\
<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0x11<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0x10<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>,\
<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0xE<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0xC<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">3</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
end
document eflags
Print entire eflags register
end
&nbsp;
define reg
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" eax:%08X ebx:%08X ecx:%08X "</span>, <span style="color: #007800;">$eax</span>, <span style="color: #007800;">$ebx</span>, <span style="color: #007800;">$ecx</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" edx:%08X eflags:%08X<span style="color: #000099; font-weight: bold;">\n</span>"</span>, <span style="color: #007800;">$edx</span>, <span style="color: #007800;">$eflags</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" esi:%08X edi:%08X esp:%08X "</span>, <span style="color: #007800;">$esi</span>, <span style="color: #007800;">$edi</span>, <span style="color: #007800;">$esp</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" ebp:%08X eip:%08X<span style="color: #000099; font-weight: bold;">\n</span>"</span>, <span style="color: #007800;">$ebp</span>, <span style="color: #007800;">$eip</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" cs:%04X ds:%04X es:%04X"</span>, <span style="color: #007800;">$cs</span>, <span style="color: #007800;">$ds</span>, <span style="color: #007800;">$es</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" fs:%04X gs:%04X ss:%04X "</span>, <span style="color: #007800;">$fs</span>, <span style="color: #007800;">$gs</span>, <span style="color: #007800;">$ss</span>
<span style="color: #7a0874; font-weight: bold;">echo</span> \033<span style="color: #7a0874; font-weight: bold;">&#91;</span>31m
flags
<span style="color: #7a0874; font-weight: bold;">echo</span> \033<span style="color: #7a0874; font-weight: bold;">&#91;</span>0m
end
document reg
Print CPU registers
end
&nbsp;
define func
info functions
end
document func
Print functions <span style="color: #000000; font-weight: bold;">in</span> target
end
&nbsp;
define var
info variables
end
document var
Print variables <span style="color: #7a0874; font-weight: bold;">&#40;</span>symbols<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">in</span> target
end
&nbsp;
define lib
info sharedlibrary
end
document lib
Print shared libraries linked to target
end
&nbsp;
define sig
info signals
end
document sig
Print signal actions <span style="color: #000000; font-weight: bold;">for</span> target
end
&nbsp;
define thread
info threads
end
document thread
Print threads <span style="color: #000000; font-weight: bold;">in</span> target
end
&nbsp;
define u
info udot
end
document u
Print kernel <span style="color: #ff0000;">'user'</span> struct <span style="color: #000000; font-weight: bold;">for</span> target
end
&nbsp;
define dis
disassemble <span style="color: #007800;">$arg0</span>
end
document dis
Disassemble address
Usage: dis addr
end
&nbsp;
<span style="color: #666666; font-style: italic;"># ________________hex/ascii dump an address______________</span>
define ascii_char
<span style="color: #666666; font-style: italic;"># thanks elaine :)</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_c</span>=<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char <span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #007800;">$_c</span> <span style="color: #000000; font-weight: bold;">&</span>lt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0x7E <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"."</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"%c"</span>, <span style="color: #007800;">$_c</span>
end
end
document ascii_char
Print the ASCII value of arg0 or <span style="color: #ff0000;">'.'</span> <span style="color: #000000; font-weight: bold;">if</span> value is unprintable
end
&nbsp;
define hex_quad
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"%02X %02X %02X %02X %02X %02X %02X %02X"</span>, \
<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> + <span style="color: #000000;">1</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>, \
<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> + <span style="color: #000000;">2</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> + <span style="color: #000000;">3</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>, \
<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> + <span style="color: #000000;">4</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> + <span style="color: #000000;">5</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>, \
<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> + <span style="color: #000000;">6</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>, <span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> + <span style="color: #000000;">7</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
end
document hex_quad
Print eight hexadecimal bytes starting at arg0
end
&nbsp;
define <span style="color: #c20cb9; font-weight: bold;">hexdump</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"%08X : "</span>, <span style="color: #007800;">$arg0</span>
hex_quad <span style="color: #007800;">$arg0</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" - "</span>
hex_quad <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+<span style="color: #000000;">8</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" "</span>
&nbsp;
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+<span style="color: #000000;">1</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+<span style="color: #000000;">2</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+<span style="color: #000000;">3</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+<span style="color: #000000;">4</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+<span style="color: #000000;">5</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+<span style="color: #000000;">6</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+<span style="color: #000000;">7</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+<span style="color: #000000;">8</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+<span style="color: #000000;">9</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+0xA<span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+0xB<span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+0xC<span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+0xD<span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+0xE<span style="color: #7a0874; font-weight: bold;">&#41;</span>
ascii_char <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span>+0xF<span style="color: #7a0874; font-weight: bold;">&#41;</span>
&nbsp;
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>"</span>
end
document <span style="color: #c20cb9; font-weight: bold;">hexdump</span>
Display a <span style="color: #000000;">16</span>-byte hex<span style="color: #000000; font-weight: bold;">/</span>ASCII dump of arg0
end
&nbsp;
<span style="color: #666666; font-style: italic;"># ________________data window__________________</span>
define ddump
<span style="color: #7a0874; font-weight: bold;">echo</span> \033<span style="color: #7a0874; font-weight: bold;">&#91;</span>36m
&nbsp;
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"[%04X:%08X]------------------------"</span>, <span style="color: #007800;">$ds</span>, <span style="color: #007800;">$data_addr</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"---------------------------------[ data]<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">echo</span> \033<span style="color: #7a0874; font-weight: bold;">&#91;</span>34m
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_count</span>=<span style="color: #000000;"></span>
<span style="color: #000000; font-weight: bold;">while</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #007800;">$_count</span> <span style="color: #000000; font-weight: bold;">&</span>lt; <span style="color: #007800;">$arg0</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_i</span>=<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$_count</span><span style="color: #000000; font-weight: bold;">*</span>0x10<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #c20cb9; font-weight: bold;">hexdump</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$data_addr</span>+<span style="color: #007800;">$_i</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_count</span>++ end end document ddump Display <span style="color: #007800;">$arg0</span> lines of <span style="color: #c20cb9; font-weight: bold;">hexdump</span> <span style="color: #000000; font-weight: bold;">for</span> address <span style="color: #007800;">$data_addr</span> end define <span style="color: #c20cb9; font-weight: bold;">dd</span> <span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0x40000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">||</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0x08000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">||</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0xBF000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$data_addr</span>=<span style="color: #007800;">$arg0</span> ddump 0x10 <span style="color: #000000; font-weight: bold;">else</span> <span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Invalid address: %08X<span style="color: #000099; font-weight: bold;">\n</span>"</span>, <span style="color: #007800;">$arg0</span> end end document <span style="color: #c20cb9; font-weight: bold;">dd</span> Display <span style="color: #000000;">16</span> lines of a hex dump <span style="color: #000000; font-weight: bold;">for</span> <span style="color: #007800;">$arg0</span> end define datawin <span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$esi</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0x40000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">||</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$esi</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0x08000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">||</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$esi</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0xBF000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$data_addr</span>=<span style="color: #007800;">$esi</span> <span style="color: #000000; font-weight: bold;">else</span> <span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$edi</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0x40000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">||</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$edi</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0x08000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">||</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$edi</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0xBF000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$data_addr</span>=<span style="color: #007800;">$edi</span> <span style="color: #000000; font-weight: bold;">else</span> <span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eax</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0x40000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">||</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eax</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0x08000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">||</span> \ <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eax</span> <span style="color: #000000; font-weight: bold;">&</span>amp; 0xBF000000<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$data_addr</span>=<span style="color: #007800;">$eax</span> <span style="color: #000000; font-weight: bold;">else</span> <span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$data_addr</span>=<span style="color: #007800;">$esp</span> end end end ddump <span style="color: #000000;">2</span> end document datawin Display esi, edi, eax, or esp <span style="color: #000000; font-weight: bold;">in</span> data window end <span style="color: #666666; font-style: italic;"># ________________process context______________ define context echo \033[36m printf "----------------------------------------" printf "---------------------------------[ regs]\n" echo \033[32m reg echo \033[36m printf "[%04X:%08X]------------------------", $ss, $esp printf "---------------------------------[stack]\n" echo \033[34m hexdump $sp+0x30 hexdump $sp+0x20 hexdump $sp+0x10 hexdump $sp datawin echo \033[36m printf "[%04X:%08X]------------------------", $cs, $eip printf "---------------------------------[ code]\n" echo \033[37m x /6i $pc echo \033[36m printf "---------------------------------------" printf "----------------------------------------\n" echo \033[0m end document context Print regs, stack, ds:esi, and disassemble cs:eip end define context-on set $SHOW_CONTEXT = 1 end document context-on Enable display of context on every program stop end define context-off set $SHOW_CONTEXT = 1 end document context-on Disable display of context on every program stop end # ________________process control______________ define n ni end document n Step one instruction end define go stepi $arg0 end document go Step # instructions end define pret finish end document pret Step out of current call end define init set $SHOW_CONTEXT = 1 set $SHOW_NEST_INSN=0 tbreak _init r end document init Run program; break on _init() end define start set $SHOW_CONTEXT = 1 set $SHOW_NEST_INSN=0 tbreak _start r end document start Run program; break on _start() end define sstart set $SHOW_CONTEXT = 1 set $SHOW_NEST_INSN=0 tbreak __libc_start_main r end document sstart Run program; break on __libc_start_main(). Useful for stripped executables. end define main set $SHOW_CONTEXT = 1 set $SHOW_NEST_INSN=0 tbreak main r end document main Run program; break on main() end # ________________eflags commands_______________ define cfc if ($eflags &amp; 1) set $eflags = $eflags&amp;~1 else set $eflags = $eflags|1 end end document cfc change Carry Flag end define cfp if (($eflags &gt;&gt; 2) &amp; 1 )</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">&</span>amp;~0x4
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">|</span>0x4
end
end
document cfp
change Carry Flag
end
&nbsp;
define cfa
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">4</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">&</span>amp;~0x10
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">|</span>0x10
end
end
document cfa
change Auxiliary Carry Flag
end
&nbsp;
define cfz
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">6</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">&</span>amp;~0x40
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">|</span>0x40
end
end
document cfz
change Zero Flag
end
&nbsp;
define cfs
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">7</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">&</span>amp;~0x80
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">|</span>0x80
end
end
document cfs
change Sign Flag
end
&nbsp;
define cft
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000;">8</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">&</span>amp;<span style="color: #000000;">100</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">|</span><span style="color: #000000;">100</span>
end
end
document cft
change Trap Flag
end
&nbsp;
define cfi
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;">9</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">&</span>amp;~0x200
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">|</span>0x200
end
end
document cfi
change Interrupt Flag
end
&nbsp;
define cfd
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt;0xA <span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">&</span>amp;~0x400
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">|</span>0x400
end
end
document cfd
change Direction Flag
end
&nbsp;
define cfo
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$eflags</span> <span style="color: #000000; font-weight: bold;">&</span>gt;<span style="color: #000000; font-weight: bold;">&</span>gt; 0xB<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #000000;">1</span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">&</span>amp;~0x800
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$eflags</span> = <span style="color: #007800;">$eflags</span><span style="color: #000000; font-weight: bold;">|</span>0x800
end
end
document cfo
change Overflow Flag
end
&nbsp;
<span style="color: #666666; font-style: italic;"># --------------------patch---------------------</span>
define <span style="color: #c20cb9; font-weight: bold;">nop</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #000000; font-weight: bold;">*</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char <span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #007800;">$arg0</span> = 0x90
end
document <span style="color: #c20cb9; font-weight: bold;">nop</span>
Patch byte at address arg0 to a NOP insn
Usage: <span style="color: #c20cb9; font-weight: bold;">nop</span> addr
end
&nbsp;
define null
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #000000; font-weight: bold;">*</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char <span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #007800;">$arg0</span> = <span style="color: #000000;"></span>
end
document null
Patch byte at address arg0 to NULL
Usage: null addr
end
&nbsp;
define int3
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #000000; font-weight: bold;">*</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char <span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #007800;">$arg0</span> = 0xCC
end
document int3
Patch byte at address arg0 to an INT3 insn
Usage: int3 addr
end
&nbsp;
<span style="color: #666666; font-style: italic;"># --------------------cflow---------------------</span>
define print_insn_type
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> == <span style="color: #000000;"></span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"UNKNOWN"</span>;
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> == <span style="color: #000000;">1</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"JMP"</span>;
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> == <span style="color: #000000;">2</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"JCC"</span>;
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> == <span style="color: #000000;">3</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"CALL"</span>;
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> == <span style="color: #000000;">4</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"RET"</span>;
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$arg0</span> == <span style="color: #000000;">5</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"INT"</span>;
end
end
document print_insn_type
This prints the human-readable mnemonic <span style="color: #000000; font-weight: bold;">for</span> the instruction typed passed <span style="color: #c20cb9; font-weight: bold;">as</span>
a parameter <span style="color: #7a0874; font-weight: bold;">&#40;</span>usually <span style="color: #007800;">$INSN_TYPE</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>.
end
&nbsp;
define get_insn_type
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$INSN_TYPE</span> = <span style="color: #000000;"></span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_byte1</span>=<span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span>unsigned char <span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#41;</span><span style="color: #007800;">$arg0</span>
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$_byte1</span> == 0x9A <span style="color: #000000; font-weight: bold;">||</span> <span style="color: #007800;">$_byte1</span> == 0xE8 <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #666666; font-style: italic;"># "call"</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$INSN_TYPE</span>=<span style="color: #000000;">3</span>
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$_byte1</span> <span style="color: #000000; font-weight: bold;">&</span>gt;= 0xE9 <span style="color: #000000; font-weight: bold;">&</span>amp;<span style="color: #000000; font-weight: bold;">&</span>amp; <span style="color: #007800;">$_byte1</span> <span style="color: #000000; font-weight: bold;">&</span>lt;= 0xEB<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #666666; font-style: italic;"># "jmp" set $INSN_TYPE=1 end if ($_byte1 &gt;= 0x70 &amp;&amp; $_byte1 &lt;= 0x7F) # "jcc" set $INSN_TYPE=2 end if ($_byte1 &gt;= 0xE0 &amp;&amp; $_byte1 &lt;= 0xE3 ) # "jcc" set $INSN_TYPE=2 end if ($_byte1 == 0xC2 || $_byte1 == 0xC3 || $_byte1 == 0xCA || $_byte1 == 0xCB || $_byte1 == 0xCF) # "ret" set $INSN_TYPE=4 end if ($_byte1 &gt;= 0xCC &amp;&amp; $_byte1 &lt;= 0xCE) # "int" set $INSN_TYPE=5 end if ($_byte1 == 0x0F ) # two-byte opcode set $_byte2=*(unsigned char *)($arg0 +1) if ($_byte2 &gt;= 0x80 &amp;&amp; $_byte2 &lt;= 0x8F) # "jcc" set $INSN_TYPE=2 end end if ($_byte1 == 0xFF ) # opcode extension set $_byte2=*(unsigned char *)($arg0 +1) set $_opext=($_byte2 &amp; 0x38) if ($_opext == 0x10 || $_opext == 0x18) # "call" set $INSN_TYPE=3 end if ($_opext == 0x20 || $_opext == 0x28) # "jmp" set $INSN_TYPE=1 end end end document get_insn_type This takes an address as a parameter and sets the global $INSN_TYPE variable to 0, 1, 2, 3, 4, 5 if the instruction at the address is unknown, a jump, a conditional jump, a call, a return, or an interrupt. end define step_to_call set $_saved_ctx = $SHOW_CONTEXT set $SHOW_CONTEXT = 0 set $SHOW_NEST_INSN=0 set logging file /dev/null set logging on set logging redirect on set $_cont = 1 while ( $_cont &gt; 0 )</span>
stepi
get_insn_type <span style="color: #007800;">$pc</span>
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$INSN_TYPE</span> == <span style="color: #000000;">3</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_cont</span> = <span style="color: #000000;"></span>
end
end
&nbsp;
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #007800;">$_saved_ctx</span> <span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;"></span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
context
<span style="color: #000000; font-weight: bold;">else</span>
x <span style="color: #000000; font-weight: bold;">/</span>i <span style="color: #007800;">$pc</span>
end
&nbsp;
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_CONTEXT</span> = <span style="color: #000000;">1</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_NEST_INSN</span>=<span style="color: #000000;"></span>
<span style="color: #000000; font-weight: bold;">set</span> logging redirect off
<span style="color: #000000; font-weight: bold;">set</span> logging off
<span style="color: #000000; font-weight: bold;">set</span> logging <span style="color: #c20cb9; font-weight: bold;">file</span> gdb.txt
end
document step_to_call
This single steps <span style="color: #000000; font-weight: bold;">until</span> it encounters a call instruction; it stops before
the call is taken.
end
&nbsp;
define trace_calls
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_CONTEXT</span> = <span style="color: #000000;"></span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_NEST_INSN</span>=<span style="color: #000000;"></span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_nest</span> = <span style="color: #000000;">1</span>
<span style="color: #000000; font-weight: bold;">set</span> listsize <span style="color: #000000;"></span>
<span style="color: #000000; font-weight: bold;">set</span> logging overwrite on
<span style="color: #000000; font-weight: bold;">set</span> logging <span style="color: #c20cb9; font-weight: bold;">file</span> ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_calls.txt
<span style="color: #000000; font-weight: bold;">set</span> logging on
<span style="color: #000000; font-weight: bold;">set</span> logging redirect on
&nbsp;
<span style="color: #000000; font-weight: bold;">while</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #007800;">$_nest</span> <span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;"></span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
get_insn_type <span style="color: #007800;">$pc</span>
&nbsp;
<span style="color: #666666; font-style: italic;"># handle nesting</span>
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$INSN_TYPE</span> == <span style="color: #000000;">3</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_nest</span> = <span style="color: #007800;">$_nest</span> + <span style="color: #000000;">1</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$INSN_TYPE</span> == <span style="color: #000000;">4</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_nest</span> = <span style="color: #007800;">$_nest</span> - <span style="color: #000000;">1</span>
end
end
&nbsp;
<span style="color: #666666; font-style: italic;"># if a call, print it</span>
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$INSN_TYPE</span> == <span style="color: #000000;">3</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$x</span> = <span style="color: #007800;">$_nest</span>
<span style="color: #000000; font-weight: bold;">while</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #007800;">$x</span> <span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;"></span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span>"</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$x</span> = <span style="color: #007800;">$x</span> - <span style="color: #000000;">1</span>
end
x <span style="color: #000000; font-weight: bold;">/</span>i <span style="color: #007800;">$pc</span>
end
&nbsp;
<span style="color: #666666; font-style: italic;">#set logging file /dev/null</span>
stepi
<span style="color: #666666; font-style: italic;">#set logging file ~/gdb_trace_calls.txt</span>
end
&nbsp;
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_CONTEXT</span> = <span style="color: #000000;">1</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_NEST_INSN</span>=<span style="color: #000000;"></span>
<span style="color: #000000; font-weight: bold;">set</span> logging redirect off
<span style="color: #000000; font-weight: bold;">set</span> logging off
<span style="color: #000000; font-weight: bold;">set</span> logging <span style="color: #c20cb9; font-weight: bold;">file</span> gdb.txt
&nbsp;
<span style="color: #666666; font-style: italic;"># clean up trace file</span>
shell <span style="color: #c20cb9; font-weight: bold;">grep</span> <span style="color: #660033;">-v</span> <span style="color: #ff0000;">' at '</span> ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_calls.txt <span style="color: #000000; font-weight: bold;">&</span>gt; ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_calls.1
shell <span style="color: #c20cb9; font-weight: bold;">grep</span> <span style="color: #660033;">-v</span> <span style="color: #ff0000;">' in '</span> ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_calls.1 <span style="color: #000000; font-weight: bold;">&</span>gt; ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_calls.txt
end
document trace_calls
Creates a runtime trace of the calls made target <span style="color: #000000; font-weight: bold;">in</span> ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_calls.txt.
Note that this is very slow because <span style="color: #c20cb9; font-weight: bold;">gdb</span> <span style="color: #ff0000;">"set redirect on"</span> does not work<span style="color: #000000; font-weight: bold;">!</span>
end
&nbsp;
define trace_run
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_CONTEXT</span> = <span style="color: #000000;"></span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_NEST_INSN</span>=<span style="color: #000000;">1</span>
<span style="color: #000000; font-weight: bold;">set</span> logging overwrite on
<span style="color: #000000; font-weight: bold;">set</span> logging <span style="color: #c20cb9; font-weight: bold;">file</span> ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_run.txt
<span style="color: #000000; font-weight: bold;">set</span> logging on
<span style="color: #000000; font-weight: bold;">set</span> logging redirect on
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_nest</span> = <span style="color: #000000;">1</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">while</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #007800;">$_nest</span> <span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;"></span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
&nbsp;
get_insn_type <span style="color: #007800;">$pc</span>
<span style="color: #666666; font-style: italic;"># jmp, jcc, or cll</span>
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$INSN_TYPE</span> == <span style="color: #000000;">3</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_nest</span> = <span style="color: #007800;">$_nest</span> + <span style="color: #000000;">1</span>
<span style="color: #000000; font-weight: bold;">else</span>
<span style="color: #666666; font-style: italic;"># ret</span>
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$INSN_TYPE</span> == <span style="color: #000000;">4</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$_nest</span> = <span style="color: #007800;">$_nest</span> - <span style="color: #000000;">1</span>
end
end
&nbsp;
stepi
end
&nbsp;
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_CONTEXT</span> = <span style="color: #000000;">1</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_NEST_INSN</span>=<span style="color: #000000;"></span>
<span style="color: #000000; font-weight: bold;">set</span> logging <span style="color: #c20cb9; font-weight: bold;">file</span> gdb.txt
<span style="color: #000000; font-weight: bold;">set</span> logging redirect off
<span style="color: #000000; font-weight: bold;">set</span> logging off
&nbsp;
<span style="color: #666666; font-style: italic;"># clean up trace file</span>
shell <span style="color: #c20cb9; font-weight: bold;">grep</span> <span style="color: #660033;">-v</span> <span style="color: #ff0000;">' at '</span> ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_run.txt <span style="color: #000000; font-weight: bold;">&</span>gt; ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_run.1
shell <span style="color: #c20cb9; font-weight: bold;">grep</span> <span style="color: #660033;">-v</span> <span style="color: #ff0000;">' in '</span> ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_run.1 <span style="color: #000000; font-weight: bold;">&</span>gt; ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_run.txt
&nbsp;
end
document trace_run
Creates a runtime trace of the target <span style="color: #000000; font-weight: bold;">in</span> ~<span style="color: #000000; font-weight: bold;">/</span>gdb_trace_run.txt. Note
that this is very slow because <span style="color: #c20cb9; font-weight: bold;">gdb</span> <span style="color: #ff0000;">"set redirect on"</span> does not work<span style="color: #000000; font-weight: bold;">!</span>
end
&nbsp;
<span style="color: #666666; font-style: italic;"># _____________________misc_____________________</span>
<span style="color: #666666; font-style: italic;"># this makes 'context' be called at every BP/step</span>
define hook-stop
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #007800;">$SHOW_CONTEXT</span> <span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;"></span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
context
end
<span style="color: #000000; font-weight: bold;">if</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span> <span style="color: #007800;">$SHOW_NEST_INSN</span> <span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;"></span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$x</span> = <span style="color: #007800;">$_nest</span>
<span style="color: #000000; font-weight: bold;">while</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #007800;">$x</span> <span style="color: #000000; font-weight: bold;">&</span>gt; <span style="color: #000000;"></span> <span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span>"</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$x</span> = <span style="color: #007800;">$x</span> - <span style="color: #000000;">1</span>
end
end
end
&nbsp;
define assemble
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Hit Ctrl-D to start, type code to assemble, hit Ctrl-D when done.<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"It is recommended to start with<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span>BITS 32<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Note that this command uses NASM (Intel syntax) to assemble.<span style="color: #000099; font-weight: bold;">\n</span>"</span>
shell <span style="color: #c20cb9; font-weight: bold;">nasm</span> <span style="color: #660033;">-f</span> bin <span style="color: #660033;">-o</span> <span style="color: #000000; font-weight: bold;">/</span>dev<span style="color: #000000; font-weight: bold;">/</span>stdout <span style="color: #000000; font-weight: bold;">/</span>dev<span style="color: #000000; font-weight: bold;">/</span>stdin <span style="color: #000000; font-weight: bold;">|</span> <span style="color: #c20cb9; font-weight: bold;">od</span> <span style="color: #660033;">-v</span> <span style="color: #660033;">-t</span> x1 <span style="color: #660033;">-w16</span> <span style="color: #660033;">-A</span> n
end
document assemble
Assemble Intel x86 instructions to binary opcodes. Uses nasm.
Usage: assemble
end
&nbsp;
define gas_asm
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Type code to assemble, hit Ctrl-D until results appear :)<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Note that this command uses GAS (AT&amp;T syntax) to assemble.<span style="color: #000099; font-weight: bold;">\n</span>"</span>
shell <span style="color: #c20cb9; font-weight: bold;">as</span> <span style="color: #660033;">-o</span> ~<span style="color: #000000; font-weight: bold;">/</span>__gdb_tmp.bin
shell objdump <span style="color: #660033;">-d</span> <span style="color: #660033;">-j</span> .text <span style="color: #660033;">--adjust-vma</span>=<span style="color: #007800;">$arg0</span> ~<span style="color: #000000; font-weight: bold;">/</span>__gdb_tmp.bin
shell <span style="color: #c20cb9; font-weight: bold;">rm</span> ~<span style="color: #000000; font-weight: bold;">/</span>__gdb_tmp.bin
end
document gas_asm
Assemble Intel x86 instructions to binary opcodes using gas and objdump
Usage: gas_asm address
end
&nbsp;
<span style="color: #666666; font-style: italic;"># !scary bp_alloc macro!</span>
<span style="color: #666666; font-style: italic;"># The idea behind this macro is to break on the following code:</span>
<span style="color: #666666; font-style: italic;"># 0x4008e0aa : sub $0xc,%esp</span>
<span style="color: #666666; font-style: italic;"># 0x4008e0ad : call 0x4008e0b2</span>
<span style="color: #666666; font-style: italic;"># 0x4008e0b2 : pop %ebx</span>
<span style="color: #666666; font-style: italic;"># 0x4008e0b3 : add $0xa3f6e,%ebx</span>
<span style="color: #666666; font-style: italic;"># At 0x4008e0b3, %ebx contains the address that has just been allocated</span>
<span style="color: #666666; font-style: italic;"># The bp_alloc macro generates this breakpoint and *should* work for</span>
<span style="color: #666666; font-style: italic;"># the forseeable future ... but if it breaks, set a breakpoint on</span>
<span style="color: #666666; font-style: italic;"># __libc_malloc and look for where where the return value gets popped.</span>
&nbsp;
define bp_alloc
tbreak <span style="color: #000000; font-weight: bold;">*</span><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #000000; font-weight: bold;">*</span>__libc_malloc + F<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #000000; font-weight: bold;">if</span> <span style="color: #007800;">$ebx</span> == <span style="color: #007800;">$arg0</span>
end
document bp_alloc
This sets a temporary breakpoint on the allocation of <span style="color: #007800;">$arg0</span>.
It works by setting a breakpoint on a specific address <span style="color: #000000; font-weight: bold;">in</span> __libc_malloc<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>.
USE WITH CAUTION <span style="color: #660033;">--</span> it is extremely platform dependent.
Usage: bp_alloc addr
end
&nbsp;
define dump_hexfile
dump ihex memory <span style="color: #007800;">$arg0</span> <span style="color: #007800;">$arg1</span> <span style="color: #007800;">$arg2</span>
end
document dump_hexfile
Write a range of memory to a <span style="color: #c20cb9; font-weight: bold;">file</span> <span style="color: #000000; font-weight: bold;">in</span> Intel ihex <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #c20cb9; font-weight: bold;">hexdump</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> format.
Usage: dump_hexfile filename start_addr end_addr
end
&nbsp;
define dump_binfile
dump memory <span style="color: #007800;">$arg0</span> <span style="color: #007800;">$arg1</span> <span style="color: #007800;">$arg2</span>
end
document dump_binfile
Write a range of memory to a binary file.
Usage: dump_binfile filename start_addr end_addr
end
&nbsp;
<span style="color: #666666; font-style: italic;"># _________________cracker tips_________________</span>
<span style="color: #666666; font-style: italic;"># The 'tips' command is used to provide tutorial-like info to the user</span>
define tips
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Tip Topic Commands:<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span>tip_display : Automatically display values on each break<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span>tip_patch : Patching binaries<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span>tip_strip : Dealing with stripped binaries<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span>tip_syntax : ATT vs Intel syntax<span style="color: #000099; font-weight: bold;">\n</span>"</span>
end
document tips
Provide a list of tips from crackers on various topics.
end
&nbsp;
define tip_patch
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" PATCHING MEMORY<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Any address can be patched using the 'set' command:<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span><span style="color: #780078;">`set ADDR = VALUE`</span> <span style="color: #000099; font-weight: bold;">\t</span>e.g. <span style="color: #780078;">`set *0x8049D6E = 0x90`</span><span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" PATCHING BINARY FILES<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Use <span style="color: #780078;">`set write`</span> in order to patch the target executable<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"directly, instead of just patching memory.<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span><span style="color: #780078;">`set write on`</span> <span style="color: #000099; font-weight: bold;">\t</span><span style="color: #780078;">`set write off`</span><span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Note that this means any patches to the code or data segments<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"will be written to the executable file. When either of these<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"commands has been issued, the file must be reloaded.<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>"</span>
end
document tip_patch
Tips on patching memory and binary files
end
&nbsp;
define tip_strip
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">" STOPPING BINARIES AT ENTRY POINT<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Stripped binaries have no symbols, and are therefore tough to<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"start automatically. To debug a stripped binary, use<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span>info file<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"to get the entry point of the file. The first few lines of<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"output will look like this:<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span>Symbols from '/tmp/a.out'<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span>Local exec file:<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span> <span style="color: #780078;">`/tmp/a.out', file type elf32-i386.\n"
printf "\t Entry point: 0x80482e0\n"
printf "Use this entry point to set an entry point:\n"
printf "\t`</span>tbreak *0x80482e0<span style="color: #780078;">`\n"
printf "The breakpoint will delete itself after the program stops as\n"
printf "the entry point.\n"
printf "\n"
end
document tip_strip
Tips on dealing with stripped binaries
end
&nbsp;
define tip_syntax
printf "\n"
printf "\t INTEL SYNTAX AT&amp;T SYNTAX\n"
printf "\tmnemonic dest, src, imm mnemonic src, dest, imm\n"
printf "\t[base+index*scale+disp] disp(base, index, scale)\n"
printf "\tregister: eax register: %%eax\n"
printf "\timmediate: 0xFF immediate: $0xFF\n"
printf "\tdereference: [addr] dereference: addr(,1)\n"
printf "\tabsolute addr: addr absolute addr: *addr\n"
printf "\tbyte insn: mov byte ptr byte insn: movb\n"
printf "\tword insn: mov word ptr word insn: movw\n"
printf "\tdword insn: mov dword ptr dword insn: movd\n"
printf "\tfar call: call far far call: lcall\n"
printf "\tfar jump: jmp far far jump: ljmp\n"
printf "\n"
printf "Note that order of operands in reversed, and that AT&amp;T syntax\n"
printf "requires that all instructions referencing memory operands \n"
printf "use an operand size suffix (b, w, d, q).\n"
printf "\n"
end
document tip_syntax
Summary of Intel and AT&amp;T syntax differences
end
&nbsp;
define tip_display
printf "\n"
printf "Any expression can be set to automatically be displayed every time\n"
printf "the target stops. The commands for this are:\n"
printf "\t`</span>display expr' : automatically display expression 'expr'<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span><span style="color: #780078;">`display' : show all displayed expressions\n"
printf "\t`</span>undisplay num' : turn off autodisplay for expression # 'num'<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"Examples:<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span><span style="color: #780078;">`display/x *(int *)$esp`</span> : print top of stack<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span><span style="color: #780078;">`display/x *(int *)($ebp+8)`</span> : print first parameter<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span><span style="color: #780078;">`display (char *)$esi`</span> : print source string<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\t</span><span style="color: #780078;">`display (char *)$edi`</span> : print destination string<span style="color: #000099; font-weight: bold;">\n</span>"</span>
<span style="color: #7a0874; font-weight: bold;">printf</span> <span style="color: #ff0000;">"<span style="color: #000099; font-weight: bold;">\n</span>"</span>
end
document tip_display
Tips on automatically displaying values when a program stops.
end
<span style="color: #666666; font-style: italic;"># __________________gdb options_________________</span>
<span style="color: #000000; font-weight: bold;">set</span> confirm off
<span style="color: #000000; font-weight: bold;">set</span> verbose off
<span style="color: #666666; font-style: italic;">#set prompt \033[01;m\033] niel@gdb $ \033[0m</span>
<span style="color: #000000; font-weight: bold;">set</span> prompt \033<span style="color: #7a0874; font-weight: bold;">&#91;</span>31mgdb $ \033<span style="color: #7a0874; font-weight: bold;">&#91;</span>0m
<span style="color: #000000; font-weight: bold;">set</span> output-radix 0x10
<span style="color: #000000; font-weight: bold;">set</span> input-radix 0x10
<span style="color: #666666; font-style: italic;"># These make gdb never pause in its output</span>
<span style="color: #000000; font-weight: bold;">set</span> height <span style="color: #000000;"></span>
<span style="color: #000000; font-weight: bold;">set</span> width <span style="color: #000000;"></span>
<span style="color: #666666; font-style: italic;"># why do these not work???</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_CONTEXT</span> = <span style="color: #000000;">1</span>
<span style="color: #000000; font-weight: bold;">set</span> <span style="color: #007800;">$SHOW_NEST_INSN</span>=<span style="color: #000000;"></span>
&nbsp;
<span style="color: #000000; font-weight: bold;">set</span> disassembly-flavor intel
&nbsp;
<span style="color: #666666; font-style: italic;">#EOF</span></pre>
        </td>
      </tr>
    </table>
  </div>
</div>