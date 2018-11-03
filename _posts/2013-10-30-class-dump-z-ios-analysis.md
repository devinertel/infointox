---
id: 123
title: Class-Dump-Z iOS Analysis
date: 2013-10-30T18:12:38+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=123
permalink: /?p=123
categories:
  - mobile
tags:
  - ARM
  - class-dump
  - clutch
  - decrypt
  - IDA
  - iOS
  - Mach-O
  - pentesting
  - RCE
---
[<img class="alignright size-full wp-image-127" alt="dump" src="http://www.infointox.net/wp-content/uploads/2013/10/dump.jpg" width="299" height="168" />](http://www.infointox.net/wp-content/uploads/2013/10/dump.jpg)Once you start looking into iOS applications one of the first things you will want to do is dump the class information. Dumping the class headers of an application will give you a lot of good information and understanding of what an application is doing. It also will be extremely valuable once you start attacking the application with cycript or your other favorite real-time hooking tool.   To do this you just need to unencrypt the binary run class-dump-z and start pouring through the results. Since iOS applications are written in Objective-C you will need a little understanding of the language. Pretty simple but a crucial step in understanding how an application works and how you can attack it.

**1. Decrypt Binary**

To dump the class information we will first need to decrypt the binary. In this example we will not decrypt the binary manually but use a tool to do the work for us. The tool of choice will be clutch. To install clutch just add <a href="http://cydia.xsellize.com" target="_blank">http://cydia.xsellize.com</a> to your cydia sources and install it from there. If you have a preferred source for clutch feel free to add it from there instead. Once clutch is installed it pretty easy to decrypt an application. On the phone just run clutch and you will see all of your installed applications. From there you just run clutch with the application name like below.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">clutch
usage: clutch <span style="color: #7a0874; font-weight: bold;">&#91;</span>application name<span style="color: #7a0874; font-weight: bold;">&#93;</span> <span style="color: #7a0874; font-weight: bold;">&#91;</span>...<span style="color: #7a0874; font-weight: bold;">&#93;</span>
Applications available: Cor.kz Free Chess
clutch Free\ Chess
Cracking Free Chess...
	<span style="color: #000000; font-weight: bold;">/</span>var<span style="color: #000000; font-weight: bold;">/</span>root<span style="color: #000000; font-weight: bold;">/</span>Documents<span style="color: #000000; font-weight: bold;">/</span>Cracked<span style="color: #000000; font-weight: bold;">/</span>Free Chess-v1.0.1.ipa</pre>
      </td>
    </tr>
  </table>
</div>

Your unencrypted application will now be in _/var/root/Documents/Cracked/_. Now we just want to install this version over the encrypted one

**2. Install Unencrypted Binary**

Now that we have an unencrypted ipa we need to install it. This can bee done many ways. Some people like to do it with <a href="http://www.i-funbox.com" target="_blank">iFunBox</a> but I will just use ipainstaller. You can install IPA Installer Console from the default BigBoss source. You will also need to install AppSync which is also available in the xsellize source above. Once you install both packages you can install the unencrypted binary. Example below.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">ipainstaller <span style="color: #660033;">-c</span> Free\ Chess-v1.0.1.ipa 
Clean installation enabled.
Will not restore any saved documents and other resources.
&nbsp;
Analyzing Free Chess-v1.0.1.ipa...
Installing Free Chess <span style="color: #7a0874; font-weight: bold;">&#40;</span>v1.0.1<span style="color: #7a0874; font-weight: bold;">&#41;</span>...
Installed Free Chess <span style="color: #7a0874; font-weight: bold;">&#40;</span>v1.0.1<span style="color: #7a0874; font-weight: bold;">&#41;</span> successfully.
Cleaning old contents of Free Chess...</pre>
      </td>
    </tr>
  </table>
</div>

**3. Class Dump**
  
We have the unencrypted version of the application installed we can now dump the class headers. To do this you will need to have Class-Dump-Z installed. I installed mine from the <a href="http://cydia.radare.org" target="_blank">cydia.radare.org</a> source. Once installed we can proceed to dump the class information. To do this go in to your application directory located in _/private/var/mobile/Applications/UID_ your UID will be different than the example below. Once in the directory you can just run class-dump-z against the binary file, I typically pipe the output to a file.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;"><span style="color: #7a0874; font-weight: bold;">cd</span> <span style="color: #000000; font-weight: bold;">/</span>private<span style="color: #000000; font-weight: bold;">/</span>var<span style="color: #000000; font-weight: bold;">/</span>mobile<span style="color: #000000; font-weight: bold;">/</span>Applications<span style="color: #000000; font-weight: bold;">/</span>25261D1C-<span style="color: #000000;">3942</span>-412C-<span style="color: #000000;">9228</span>-33B253307EA0<span style="color: #000000; font-weight: bold;">/</span>Free\ Chess.app
class-dump-z Free\ Chess <span style="color: #000000; font-weight: bold;">&</span>gt; class-dump.txt</pre>
      </td>
    </tr>
  </table>
</div>

You now have all the class headers of the application in a txt file. From here you can analyze them on the phone or pull it down start pouring though it. You want to look for interesting functions you may want to hook when doing real-time analysis. Things to look for a will depend on the application but some ideas are remove passwords, decrypt data, intercept traffic or remove jail break detection. Within the class dump headers <span style="line-height: 1.5em;">you will see @interface and @protocol. @interface will have the declaration of methods and the properties and @protocol will related methods it may use. Typically we will want to look for @interface areas.  From the chess app I dumped, below are some interesting functions to get an idea of things you will see.</span>

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="c" style="font-family:monospace;">__attribute__<span style="color: #009900;">&#40;</span><span style="color: #009900;">&#40;</span>visibility<span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"hidden"</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span>
@interface PromotionView <span style="color: #339933;">:</span> SafeTouchesView <span style="color: #009900;">&#123;</span>
@private
	NSMutableArray<span style="color: #339933;">*</span> arrayImages<span style="color: #339933;">;</span>
	BOOL backPressed<span style="color: #339933;">;</span>
	CGRect backRect<span style="color: #339933;">;</span>
	CGRect arrayRects<span style="color: #009900;">&#91;</span><span style="color: #0000dd;">5</span><span style="color: #009900;">&#93;</span><span style="color: #339933;">;</span>
	BOOL _hitQueen<span style="color: #339933;">;</span>
	BOOL _hitBishop<span style="color: #339933;">;</span>
	BOOL _hitKnight<span style="color: #339933;">;</span>
	BOOL _hitRook<span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span></pre>
      </td>
    </tr>
  </table>
</div>

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="c" style="font-family:monospace;">__attribute__<span style="color: #009900;">&#40;</span><span style="color: #009900;">&#40;</span>visibility<span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"hidden"</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span>
@interface AdvertisingView <span style="color: #339933;">:</span> UIView <span style="color: #009900;">&#123;</span>
@private
	<span style="color: #993333;">int</span> m_nCurrentScene<span style="color: #339933;">;</span>
	<span style="color: #993333;">int</span> m_nCurrentSoftAd<span style="color: #339933;">;</span>
	bool m_bDrawDef<span style="color: #339933;">;</span>
	NSTimer<span style="color: #339933;">*</span> m_Timer<span style="color: #339933;">;</span>
	NSTimer<span style="color: #339933;">*</span> m_TimerSoftAd<span style="color: #339933;">;</span>
	NSMutableData<span style="color: #339933;">*</span> _receivedData<span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>initWithFrame<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>CGRect<span style="color: #009900;">&#41;</span>frame<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>copyResourceImageToDocumentFolder<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>copyResourceConfigToDocumentFolder<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>autoDownloadConfig<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>saveAdImage<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>image<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>loadConfig<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">int</span><span style="color: #009900;">&#41;</span>config<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>loadImage<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>isFinishThread<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>switchSoftAd<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>switchScene<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>doTransition<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>basic_string<span style="color: #339933;">&</span>lt<span style="color: #339933;">;</span><span style="color: #993333;">char</span><span style="color: #339933;">,</span>std<span style="color: #339933;">::</span><span style="color: #202020;">char_traits</span><span style="color: #339933;">,</span>std<span style="color: #339933;">::</span><span style="color: #202020;">allocator</span> <span style="color: #339933;">&</span>gt<span style="color: #339933;">;</span><span style="color: #009900;">&#41;</span>transition<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>callLinkCountProduct<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>touchesBegan<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>began withEvent<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>event<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>touchesEnded<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>ended withEvent<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>event<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>connection<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>connection didReceiveResponse<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>response<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>connection<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>connection didReceiveData<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>data<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>connection<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>connection didFailWithError<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>error<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>connectionDidFinishLoading<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>connection<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>drawRect<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>CGRect<span style="color: #009900;">&#41;</span>rect<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>drawView<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>CGRect<span style="color: #009900;">&#41;</span>view<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>dealloc<span style="color: #339933;">;</span>
@end</pre>
      </td>
    </tr>
  </table>
</div>

This is not the best example since you basically will just win at chess but it gives you a good idea of what you will see in the class headers and how much it helps you understand the application for further testing.

**References:**

  * <a href="http://www.securitytube.net/video/6642" target="_blank">http://www.securitytube.net/video/6642</a>
  * <a href="https://code.google.com/p/networkpx/downloads/detail?name=class-dump-z" target="_blank">https://code.google.com/p/networkpx/downloads/detail?name=class-dump-z</a>

&nbsp;

<p style="text-align: center;">
  <strong>*****UPDATE 12/29/13*****</strong>
</p>

<p style="text-align: left;">
  A new version of clutch has been released for armv7, armv7s, & arm64. You can download the latest release from <a href="https://github.com/KJCracks/Clutch-dl/" target="_blank">https://github.com/KJCracks/Clutch-dl/</a> . Once you download the git file simply scp it over to the phone and give it execute permission. Below is the output when run against an app. All other steps are the same once it creates the unencrypted ipa file in <em>/var/root/Documents/Cracked/.</em>
</p>

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;"><span style="color: #c20cb9; font-weight: bold;">chmod</span> +x Clutch-<span style="color: #000000;">1.3</span>-<span style="color: #000000;">3.2</span>-git4
.<span style="color: #000000; font-weight: bold;">/</span>Clutch-<span style="color: #000000;">1.3</span>-<span style="color: #000000;">3.2</span>-git4 <span style="color: #660033;">-C</span> Cor.kz
You<span style="color: #000000; font-weight: bold;">\'</span>re using a Clutch development build, checking <span style="color: #000000; font-weight: bold;">for</span> updates..
Your version of Clutch is up to <span style="color: #c20cb9; font-weight: bold;">date</span><span style="color: #000000; font-weight: bold;">!</span>
Downloading config files..
Clutch configuration
===============
ProgressBar
 - Do you want to show progress bar <span style="color: #7a0874; font-weight: bold;">&#40;</span>YES<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #c20cb9; font-weight: bold;">yes</span>
MetadataEmail
 - What email should we <span style="color: #c20cb9; font-weight: bold;">patch</span> <span style="color: #000000; font-weight: bold;">in</span> metadata? <span style="color: #7a0874; font-weight: bold;">&#40;</span>steve<span style="color: #000000; font-weight: bold;">@</span>rim.jobs<span style="color: #7a0874; font-weight: bold;">&#41;</span> bah<span style="color: #000000; font-weight: bold;">@</span>bah.com
RemoveMetadata
 - Do you want to remove metadata <span style="color: #7a0874; font-weight: bold;">&#40;</span>NO<span style="color: #7a0874; font-weight: bold;">&#41;</span> 
Using default value..
VerboseLogging
 - Do you want to <span style="color: #7a0874; font-weight: bold;">enable</span> verbose logging <span style="color: #7a0874; font-weight: bold;">&#40;</span>NO<span style="color: #7a0874; font-weight: bold;">&#41;</span> 
Using default value..
NumberBasedMenu
 - Do you want a number based menu? <span style="color: #7a0874; font-weight: bold;">&#40;</span>NO<span style="color: #7a0874; font-weight: bold;">&#41;</span> 
Using default value..
Not enabled entry MetadataPurchaseDate
CheckMinOS
 - Do you want to check minimum OS? <span style="color: #7a0874; font-weight: bold;">&#40;</span>adds minimum OS version to <span style="color: #c20cb9; font-weight: bold;">file</span> name<span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #7a0874; font-weight: bold;">&#40;</span>NO<span style="color: #7a0874; font-weight: bold;">&#41;</span> 
Using default value..
CrackerName
 - What<span style="color: #000000; font-weight: bold;">\'</span>s your name? <span style="color: #7a0874; font-weight: bold;">&#40;</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> <span style="color: #7a0874; font-weight: bold;">test</span>
CreditFile
 - Do you want a credit <span style="color: #c20cb9; font-weight: bold;">file</span> <span style="color: #000000; font-weight: bold;">in</span> the IPA? <span style="color: #7a0874; font-weight: bold;">&#40;</span>NO<span style="color: #7a0874; font-weight: bold;">&#41;</span> 
Using default value..
ListWithDisplayName
 - Do you want a show the application display name on lists? <span style="color: #7a0874; font-weight: bold;">&#40;</span>NO<span style="color: #7a0874; font-weight: bold;">&#41;</span> 
Using default value..
CompressionLevel
 - What <span style="color: #000000; font-weight: bold;">do</span> you want the compression level to be? Use <span style="color: #660033;">-1</span> <span style="color: #000000; font-weight: bold;">for</span> default compression <span style="color: #7a0874; font-weight: bold;">&#40;</span>-<span style="color: #000000;">1</span><span style="color: #7a0874; font-weight: bold;">&#41;</span> 
Using default value..
FilenameCredit
 - Do you want your name <span style="color: #000000; font-weight: bold;">in</span> the IPA<span style="color: #000000; font-weight: bold;">\'</span>s name? <span style="color: #7a0874; font-weight: bold;">&#40;</span>NO<span style="color: #7a0874; font-weight: bold;">&#41;</span> 
Using default value..
CheckMetadata
 - Do you want to check metadata <span style="color: #7a0874; font-weight: bold;">&#40;</span>NO<span style="color: #7a0874; font-weight: bold;">&#41;</span> 
Using default value..</pre>
      </td>
    </tr>
  </table>
</div>