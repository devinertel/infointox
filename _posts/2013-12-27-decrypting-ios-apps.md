---
id: 114
title: Decrypting iOS Apps
date: 2013-12-27T22:41:27+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=114
permalink: /?p=114
categories:
  - mobile
tags:
  - ARM
  - Cracking
  - decrypt
  - gdb
  - IDA
  - iOS
  - Mach-O
  - mobile
---
[<img class="alignright  wp-image-119" alt="decrypt iOS apps" src="http://www.infointox.net/wp-content/uploads/2013/12/iphone-113-hack-300x300.jpg" width="134" height="134" srcset="http://www.infointox.net/wp-content/uploads/2013/12/iphone-113-hack-300x300.jpg 300w, http://www.infointox.net/wp-content/uploads/2013/12/iphone-113-hack-150x150.jpg 150w, http://www.infointox.net/wp-content/uploads/2013/12/iphone-113-hack.jpg 400w" sizes="(max-width: 134px) 100vw, 134px" />](http://www.infointox.net/wp-content/uploads/2013/12/iphone-113-hack.jpg)I&#8217;m going to walk through the steps it takes to decrypt an iOS app. Since iOS encrypts all of its apps it makes it basically impossible to do any static analysis on an app without decrypting first. If you want to be able to check out an app within IDA you will first have to decrypt it and repackage it. At a high level we will make iOS decrypt the binary. While its decrypted we will extract the decrypted portions with GDB. We will then take the unencrypted portion and patch it over the encrypted portion of the binary with dd.  This post doesn&#8217;t talk about the reversing process once you get the app to IDA but what you need to do to start reversing an app in IDA and what you need to do to get the reversed app running on the phone again. And it goes without saying this must all be done on a phone that you have root on.

**1. Extract the application**
  
iOS apps live in _/private/var/mobile/Applications/UID_ where the UID is the universally unique identifier of the app. You will have to cd into the different directories until you find what the app you are looking to analyze. My example below.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;"><span style="color: #7a0874; font-weight: bold;">cd</span> <span style="color: #000000; font-weight: bold;">/</span>private<span style="color: #000000; font-weight: bold;">/</span>var<span style="color: #000000; font-weight: bold;">/</span>mobile<span style="color: #000000; font-weight: bold;">/</span>Applications<span style="color: #000000; font-weight: bold;">/</span>A54D09EF-E8CD-41AD-A10E-090302B62562<span style="color: #000000; font-weight: bold;">/</span></pre>
      </td>
    </tr>
  </table>
</div>

While its a bit overkill, once you find the correct directory you can just scp down the whole .app folder down to your machine.

**2. Find the binary**
  
Now we have the whole working directory we just need to find the binary that we will be decrypting. This can be done with otool that comes with Xcode. Just run otool within the directory you just pulled down. As you see below have an have an ARM binary. This will be the binary we will want to decrypt.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">otool <span style="color: #660033;">-arch</span> all <span style="color: #660033;">-Vh</span> Free\ Chess 
Free Chess:
Mach header
      magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
   MH_MAGIC     ARM         V6  0x00     EXECUTE    <span style="color: #000000;">23</span>       <span style="color: #000000;">2672</span>   NOUNDEFS DYLDLINK TWOLEVEL BINDS_TO_WEAK</pre>
      </td>
    </tr>
  </table>
</div>

**3. Find encrypted area**
  
We now need to find what area of the binary is encrypted. We will use this information to patch the decrypted portion over the encrypted portion using dd.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">otool <span style="color: #660033;">-arch</span> all <span style="color: #660033;">-Vl</span> Free\ Chess <span style="color: #000000; font-weight: bold;">|</span> <span style="color: #c20cb9; font-weight: bold;">grep</span> <span style="color: #660033;">-A5</span> LC_ENCRYP
          cmd LC_ENCRYPTION_INFO
      cmdsize <span style="color: #000000;">20</span>
    cryptoff  <span style="color: #000000;">4096</span>
    cryptsize <span style="color: #000000;">192512</span>
    cryptid   <span style="color: #000000;">1</span>
Load <span style="color: #7a0874; font-weight: bold;">command</span> <span style="color: #000000;">10</span></pre>
      </td>
    </tr>
  </table>
</div>

As you can see cryptoff is 4096 which is the start of the encrypted portion and cryptsize is 192512 showing the size of the encrypted area. Note cryptid which is 1. This means the application is encrypted. We will flip this later so iOS thinks it decrypted and won&#8217;t try to decrypt the app again.

Running the below command will show you the LC_SEGMENT load command within the TEXT segment.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">otool <span style="color: #660033;">-arch</span> all <span style="color: #660033;">-Vl</span> Free\ Chess 
Free Chess:
Load <span style="color: #7a0874; font-weight: bold;">command</span> <span style="color: #000000;"></span>
      cmd LC_SEGMENT
  cmdsize <span style="color: #000000;">56</span>
  segname __PAGEZERO
   vmaddr 0x00000000
   vmsize 0x00001000
  fileoff <span style="color: #000000;"></span>
 filesize <span style="color: #000000;"></span>
  maxprot <span style="color: #660033;">---</span>
 initprot <span style="color: #660033;">---</span>
   nsects <span style="color: #000000;"></span>
    flags <span style="color: #7a0874; font-weight: bold;">&#40;</span>none<span style="color: #7a0874; font-weight: bold;">&#41;</span>
Load <span style="color: #7a0874; font-weight: bold;">command</span> <span style="color: #000000;">1</span>
      cmd LC_SEGMENT
  cmdsize <span style="color: #000000;">328</span>
  segname __TEXT
   vmaddr 0x00001000
   vmsize 0x00030000
  fileoff <span style="color: #000000;"></span>
 filesize <span style="color: #000000;">196608</span>
  maxprot r-x
 initprot r-x
   nsects <span style="color: #000000;">4</span>
    flags <span style="color: #7a0874; font-weight: bold;">&#40;</span>none<span style="color: #7a0874; font-weight: bold;">&#41;</span></pre>
      </td>
    </tr>
  </table>
</div>

So we see the app is starting at 0x00001000 and we know our encrypted data starts at 4096 from cryptoff. Pull out a hex calculator and you will find 4096 is 0x1000. We now have all the information we need about the encrypted portion of the app so we can extract it once its unencrypted and patch it over with the encrypted portion. We know the app starts at 0x00001000 and and the encrypted portion starts at 0x1000 within the app. This is why will now add these together to get where the encrypted portion starts within the app. 0x00001000 + 0x1000 = 0x2000. 0x200 is where we will want to begin to extract a binary dump.

**4. Extract unencrypted data**
  
Now that we know what portions of the app we want to extract we need to load the app on the phone so we can get it to decrypt. We can just run that app and attached to the pid to ensure its been decrypted. Once the app is unencrypted we can dump the portions we need to patch the binary. Before you begin fooling around with GDB on your phone uninstall the default one that comes with Cydia. I always just add the <a href="http://cydia.radare.org" target="_blank">Radare</a> repo and install GDB from there. At the time of writing this it was at version 1708. I like radare as a cydia source because it has other useful tools you will be using down the road.

_Note: Some of you may get the &#8220;Illegal instruction: 4&#8221; when trying to run within GDB. This is due to old compiled binaries. You can patch the binary with the below command. You have to install sed and ldid(Link Identity Editor)._

 `sed -i'' 's/\x00\x30\x93\xe4/\x00\x30\x93\xe5/g;s/\x00\x30\xd3\xe4/\x00\x30\xd3\xe5/g;' ios_binary<br />
ldid -s ios_binary<br />
` 

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;"><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #c20cb9; font-weight: bold;">gdb</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> attach Free\ Chess.2545 
Attaching to process <span style="color: #000000;">2545</span>.
Reading symbols <span style="color: #000000; font-weight: bold;">for</span> shared libraries . <span style="color: #000000; font-weight: bold;">done</span>
Reading symbols <span style="color: #000000; font-weight: bold;">for</span> shared libraries...................................................... <span style="color: #000000; font-weight: bold;">done</span>
0x3a468e30 <span style="color: #000000; font-weight: bold;">in</span> mach_msg_trap <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #c20cb9; font-weight: bold;">gdb</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> dump binary memory memorydump.bin 0x2000 <span style="color: #7a0874; font-weight: bold;">&#40;</span>0x2000 + <span style="color: #000000;">192512</span><span style="color: #7a0874; font-weight: bold;">&#41;</span>
<span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #c20cb9; font-weight: bold;">gdb</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> q 
The program is running.  Quit anyway <span style="color: #7a0874; font-weight: bold;">&#40;</span>and detach it<span style="color: #7a0874; font-weight: bold;">&#41;</span>? <span style="color: #7a0874; font-weight: bold;">&#40;</span>y or n<span style="color: #7a0874; font-weight: bold;">&#41;</span> y
Detaching from process <span style="color: #000000;">2545</span>.</pre>
      </td>
    </tr>
  </table>
</div>

As you can see we just load GDB type the app you want to attach to and just tab and it will complete the correct PID. The example above is PID 2545. The line below is what you want to pay attention to.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;"><span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #c20cb9; font-weight: bold;">gdb</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> dump binary memory memorydump.bin 0x2000 <span style="color: #7a0874; font-weight: bold;">&#40;</span>0x2000 + <span style="color: #000000;">192512</span><span style="color: #7a0874; font-weight: bold;">&#41;</span></pre>
      </td>
    </tr>
  </table>
</div>

Dumped memory from 0x2000 to 0x2000 + 192512.

**5. Patch Binary**
  
Now we will want to patch our binary. We can scp our recent memory dump down to our directory on our machine and patch the encrypted binary we already have. This can be done with the below command.  Seek is set to cryptoff and patched_chess is the name of the file I copied the original binary to.

_Note: Always backup your binaries before any modifications. _

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;"><span style="color: #c20cb9; font-weight: bold;">dd</span> <span style="color: #007800;">bs</span>=<span style="color: #000000;">1</span> <span style="color: #007800;">seek</span>=<span style="color: #000000;">4096</span> <span style="color: #007800;">conv</span>=notrunc <span style="color: #007800;">if</span>=memorydump.bin <span style="color: #007800;">of</span>=patched_chess
<span style="color: #000000;">192512</span>+<span style="color: #000000;"></span> records <span style="color: #000000; font-weight: bold;">in</span>
<span style="color: #000000;">192512</span>+<span style="color: #000000;"></span> records out
<span style="color: #000000;">192512</span> bytes transferred <span style="color: #000000; font-weight: bold;">in</span> <span style="color: #000000;">0.401887</span> secs <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #000000;">479020</span> bytes<span style="color: #000000; font-weight: bold;">/</span>sec<span style="color: #7a0874; font-weight: bold;">&#41;</span></pre>
      </td>
    </tr>
  </table>
</div>

You should now have a unencrypted file of the same size as the original. You can now name this new file with original binary name. You are all set to plug it into IDA and look under the hood. Once you are done reversing and making any hex editor changes you can scp the binary back to the phone. The one thing you want to do before is to disable load encryption.

**6. Disable Load Encryption**
  
Within the binary there is a simple flag that tell iOS to decrypt the app on load. Now that our binary is already decrypted we need to turn that off. As you see below cryptid is 1. We will want to change that to 0.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">otool <span style="color: #660033;">-arch</span> all <span style="color: #660033;">-Vl</span> Free\ Chess <span style="color: #000000; font-weight: bold;">|</span> <span style="color: #c20cb9; font-weight: bold;">grep</span> <span style="color: #660033;">-A5</span> LC_ENCRYP
          cmd LC_ENCRYPTION_INFO
      cmdsize <span style="color: #000000;">20</span>
    cryptoff  <span style="color: #000000;">4096</span>
    cryptsize <span style="color: #000000;">192512</span>
    cryptid   <span style="color: #000000;">1</span>
Load <span style="color: #7a0874; font-weight: bold;">command</span> <span style="color: #000000;">10</span></pre>
      </td>
    </tr>
  </table>
</div>

I typically just eyeball it with a hex editor but a nice way to see this is with <a href="http://sourceforge.net/projects/machoview/files/" target="_blank">MachOView</a>. In MachOView you can go under **Load Commands > LC\_ENCRYPTION\_INFO** to view the offset of CryptID.

[<img class="aligncenter size-full wp-image-115" alt="machoview" src="http://www.infointox.net/wp-content/uploads/2013/12/machoview.png" width="898" height="410" srcset="http://www.infointox.net/wp-content/uploads/2013/12/machoview.png 898w, http://www.infointox.net/wp-content/uploads/2013/12/machoview-300x136.png 300w" sizes="(max-width: 898px) 100vw, 898px" />](http://www.infointox.net/wp-content/uploads/2013/12/machoview.png)

Now we know the location of cryptid we can open the binary in a hex editor. I use [Hex Fiend](http://ridiculousfish.com/hexfiend/) in this example. We know the offset already but I always just search for the first instance /System/Library/Frameworks and just above it you will see bytes 01. It typically stands out pretty easily. Now in the hex editor flip this to 00. As you see in the screenshot it is at the offset we found in MachOView and just above /System/Library/Frameworks. So you can pick your poison. Make a backup before you save the changes.

[<img class="aligncenter size-full wp-image-116" alt="hexfiend" src="http://www.infointox.net/wp-content/uploads/2013/12/hexfiend.png" width="891" height="475" srcset="http://www.infointox.net/wp-content/uploads/2013/12/hexfiend.png 891w, http://www.infointox.net/wp-content/uploads/2013/12/hexfiend-300x159.png 300w" sizes="(max-width: 891px) 100vw, 891px" />](http://www.infointox.net/wp-content/uploads/2013/12/hexfiend.png)

Now once you flipped the bit you can verify this be running the same command. As you can see cryptid is now 0.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">otool <span style="color: #660033;">-arch</span> all <span style="color: #660033;">-Vl</span> Free\ Chess <span style="color: #000000; font-weight: bold;">|</span> <span style="color: #c20cb9; font-weight: bold;">grep</span> <span style="color: #660033;">-A5</span> LC_ENCRYP
          cmd LC_ENCRYPTION_INFO
      cmdsize <span style="color: #000000;">20</span>
    cryptoff  <span style="color: #000000;">4096</span>
    cryptsize <span style="color: #000000;">192512</span>
    cryptid   <span style="color: #000000;"></span>
Load <span style="color: #7a0874; font-weight: bold;">command</span> <span style="color: #000000;">10</span></pre>
      </td>
    </tr>
  </table>
</div>

Now that you have made all the reversing changes you needed and flipped cryptid you can now upload this new shiny binary into the working directory on the phone. Make sure the app is not running and to run ldid.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">ldid <span style="color: #660033;">-s</span> Free\ Chess</pre>
      </td>
    </tr>
  </table>
</div>

**References:**
  
<a href="http://www.sensepost.com/blog/6254.html" target="_blank">http://www.sensepost.com/blog/6254.html</a>
  
<a href="http://iphonesdkdev.blogspot.com" target="_blank">http://iphonesdkdev.blogspot.com</a>
  
<a href="http://my.opera.com/iphonedev/blog/2008/08/11/how-to-crack-apple-apps" target="_blank">http://my.opera.com/iphonedev/blog/2008/08/11/how-to-crack-apple-apps</a>
  
<a href="http://dvlabs.tippingpoint.com/blog/2009/03/06/reverse-engineering-iphone-appstore-binaries" target="_blank">http://dvlabs.tippingpoint.com/blog/2009/03/06/reverse-engineering-iphone-appstore-binaries</a>
  
<a href="http://www.mandalorian.com/2013/05/decrypting-ios-binaries/" target="_blank">http://www.mandalorian.com/2013/05/decrypting-ios-binaries/</a>
  
<a href="http://lightbulbone.com/post/27887705317/reversing-ios-applications-part-1" target="_blank">http://lightbulbone.com/post/27887705317/reversing-ios-applications-part-1</a>
  
<a href="http://www.pod2g.org/2012/02/working-gnu-debugger-on-ios-43.html" target="_blank">http://www.pod2g.org/2012/02/working-gnu-debugger-on-ios-43.html</a>

**Tools:**

  * otool (from  Xcode or cydia) &#8211;  object file displaying tool
  * ldid (from cydia) &#8211; Link Identity Editor
  * gdb (from <a href="http://cydia.radare.org/debs/" target="_blank">cydia radar repo</a>) &#8211; GNU Debugger for iOS build 1708
  * <a href="http://sourceforge.net/projects/machoview/files/" target="_blank">MachOView</a> &#8211; visual Mach-O file browser
  * <a href="http://ridiculousfish.com/hexfiend/" target="_blank">Hex Fiend</a> &#8211; open source hex editor for Mac OS X