---
id: 50
title: JNLP/JAR Hacking
date: 2010-05-26T17:57:28+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=50
permalink: /?p=50
categories:
  - Uncategorized
tags:
  - Jar
  - Java
  - JNLP
---
[<img class="alignright  size-medium wp-image-51" title="Java" src="http://www.infointox.net/wp-content/uploads/2010/05/Java-Evil-Edition-orfjackal_net-lores-e1274829730525-186x300.png" alt="" width="186" height="300" srcset="http://www.infointox.net/wp-content/uploads/2010/05/Java-Evil-Edition-orfjackal_net-lores-e1274829730525-186x300.png 186w, http://www.infointox.net/wp-content/uploads/2010/05/Java-Evil-Edition-orfjackal_net-lores-e1274829730525.png 288w" sizes="(max-width: 186px) 100vw, 186px" />](http://www.infointox.net/wp-content/uploads/2010/05/Java-Evil-Edition-orfjackal_net-lores.png)
  
JNLP is essentially Java&#8217;s version of Flash. It runs as a web link and caches a copy locally. Now since we can decompile Java this leaves room for attacks similar to the old fashion flash game cheating. Many of these Jar files are stored in common temp directories. Once the Jar file is located the modifications can begin.

Nothing in here is really hacking at all, I just wanted to combine some known Java functionality and craft it into something that could be used in an attack. There are two main ways we can go about this. One is to decompile the code and recompile with the changes or actually do binary manipulation in IDA. I will not talk about any details on binary manipulation.

First lets talk about if a Jar file is signed. When the Jar file is signed, any modifications you try to make to the code will fail on an MD5 check. To get around this you can make your own keys and sign it with them after you have made your modifications.

**JAR Signing**

_Note: These tools are located in the JDK bin section.
  
_ 

1. Create Your Own Keys

 `keytool -genkey -keystore mykeystore -alias test`

2. Sign Modified JAR File
  
 ``

 `jarsigner -keystore mykeystore –signedjar app-signed.jar new-app.jar test`

3. Verify Signing
  
 ``

 `jarsigner -verify -keystore mykeystore  app-signed.jar`

**Decompiling JARs**

This first method of decompiling JAR file is the easiest to modify code.  Some problems may arise depending on java versions and libraries.  You can extract the JAR file with many programs such as 7zip. The extracted copy is where you will replace your modified java files. Remember to remove the META-INF data if you plan to sign the JAR with your own keys.

You can now decompile your JAR file to view the code. In the below example I am using <a title="http://java.decompiler.free.fr/" href="http://java.decompiler.free.fr/" target="_blank">JD-GUI</a> .  Another nice decompiler is <a title="JAD" href="http://www.varaneckas.com/jad" target="_blank">JAD</a>.  Once you decompile you can view and export the specific Java files you would like to modify.

[<img class="aligncenter size-medium wp-image-53" title="jdgui" src="http://www.infointox.net/wp-content/uploads/2010/05/jdgui-300x183.png" alt="" width="300" height="183" srcset="http://www.infointox.net/wp-content/uploads/2010/05/jdgui-300x183.png 300w, http://www.infointox.net/wp-content/uploads/2010/05/jdgui.png 977w" sizes="(max-width: 300px) 100vw, 300px" />](http://www.infointox.net/wp-content/uploads/2010/05/jdgui.png)

Make your modifications and replace the original with your modified version in the area you extracted the JAR to.

Compile and Sign
  
`<br />
javac -cp .;c:\Progra~1\Java\jdk1.5.0_17\lib;C:\app newfile.java`

**JAR Binary Modifications**

This section is a little trickier since you have make changes in a binary disassembled form. To make it easier read though the code with a java decompiler and look for functions you want to modify.  Once you know what you want to modify search for those functions in the disassembler to find the correct offset. Once the offset is found, you can modify the JAR with a hex editor.

_Note: This document does not discuss any reverse engineering techniques._ 

1.  Find function you would like to modify and load into IDA.

[<img class="aligncenter size-medium wp-image-54" title="idajar" src="http://www.infointox.net/wp-content/uploads/2010/05/idajar-300x233.png" alt="" width="300" height="233" srcset="http://www.infointox.net/wp-content/uploads/2010/05/idajar-300x233.png 300w, http://www.infointox.net/wp-content/uploads/2010/05/idajar.png 705w" sizes="(max-width: 300px) 100vw, 300px" />](http://www.infointox.net/wp-content/uploads/2010/05/idajar.png)

2.   Now you have find the offset of this function in IDA. This offset will be used to  modify the binary with a hex editor.  Open the Jar file in a hex editor and search for that offset to make your changes. In the example below I used <a title="http://mh-nexus.de/en/hxd" href="http://mh-nexus.de/en/hxd" target="_blank">HxD</a> . Check out the security tube link below for more details.

[<img class="aligncenter size-medium wp-image-55" title="hexedit" src="http://www.infointox.net/wp-content/uploads/2010/05/hexedit-300x179.png" alt="" width="300" height="179" srcset="http://www.infointox.net/wp-content/uploads/2010/05/hexedit-300x179.png 300w, http://www.infointox.net/wp-content/uploads/2010/05/hexedit.png 977w" sizes="(max-width: 300px) 100vw, 300px" />](http://www.infointox.net/wp-content/uploads/2010/05/hexedit.png)

**References:**

&#8211;        [http://www.owasp.org/index.php/Signing\_jar\_files\_with\_jarsigner](http://www.owasp.org/index.php/Signing_jar_files_with_jarsigner)

&#8211;        <http://java.sun.com/j2se/1.5.0/docs/tooldocs/solaris/jarsigner.html>

&#8211;        <http://java.sun.com/j2se/1.5.0/docs/tooldocs/solaris/keytool.html>

&#8211;        <a title="http://www.securitytube.net/Java-and-JNLP-Application-Hacking-video.aspx" href="http://www.securitytube.net/Java-and-JNLP-Application-Hacking-video.aspx" target="_blank">http://www.securitytube.net/Java-and-JNLP-Application-Hacking-video.aspx</a>