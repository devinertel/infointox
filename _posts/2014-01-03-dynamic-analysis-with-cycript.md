---
id: 125
title: Dynamic Analysis with Cycript
date: 2014-01-03T12:19:27+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=125
permalink: /?p=125
categories:
  - mobile
tags:
  - ARM
  - class-dump
  - Cracking
  - cycript
  - iOS
  - Mach-O
  - MobileSubstrate
---
[<img class="alignright size-full wp-image-129" alt="static_dynamic" src="http://www.infointox.net/wp-content/uploads/2014/01/static-dynamic1.jpg" width="198" height="198" srcset="http://www.infointox.net/wp-content/uploads/2014/01/static-dynamic1.jpg 198w, http://www.infointox.net/wp-content/uploads/2014/01/static-dynamic1-150x150.jpg 150w" sizes="(max-width: 198px) 100vw, 198px" />](http://www.infointox.net/wp-content/uploads/2014/01/static-dynamic1.jpg)Cycript is a console based application that is a blend of Objective-C and JavaScript. Cycript is very useful for dynamic analysis of iOS applications.  What you can do with it is hook a running application on the phone and dynamically change parameters, rewrite objects and intercept communication within the application.  This can be done by injecting a static library or injecting as the app is running. Cycript does depend on MobileSubstrate so be sure to install that within Cydia. To get started with Cycript I&#8217;m going to walk through getting behind a password protected app. I&#8217;m not going to disclose the app as I don&#8217;t want to disclose any vulnerabilities. This is example should universally apply , there are ton of apps that state you can &#8220;securely&#8221; lock things in them just download one from the app store as I did.

**1. Hook the application**
  
Open the application on the phone and find the PID of the application you are going to hook.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;"><span style="color: #c20cb9; font-weight: bold;">ps</span> <span style="color: #660033;">-A</span> <span style="color: #000000; font-weight: bold;">|</span><span style="color: #c20cb9; font-weight: bold;">grep</span> 
cycript <span style="color: #660033;">-p</span></pre>
      </td>
    </tr>
  </table>
</div>

**2. Find the Lock Screen Controller**
  
The app should be sitting at the lock screen so we will want to find out the controller that is current at the time as this is the one we will want to manipulate.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="javascript" style="font-family:monospace;">cy# UIApp.<span style="color: #660066;">keyWindow</span>.<span style="color: #660066;">rootViewController</span>
<span style="color: #339933;">@</span><span style="color: #3366CC;">""</span>
cy# LockScreen <span style="color: #339933;">=</span> UIApp.<span style="color: #660066;">keyWindow</span>.<span style="color: #660066;">rootViewController</span>
<span style="color: #339933;">@</span><span style="color: #3366CC;">""</span>
cy# LockScreen.<span style="color: #660066;">visibleViewController</span>
<span style="color: #009900;">&#91;</span><span style="color: #339933;">@</span><span style="color: #3366CC;">"&lt;PatternLockViewController: 0x1d88d4f0"</span><span style="color: #009900;">&#93;</span></pre>
      </td>
    </tr>
  </table>
</div>

As you can see by the last line we now know the name of the lock screen view controller is &#8220;PatternLockViewController&#8221;. This is the one we will want to look into. In the example above I created a variable named LockScreen to the current ViewController. This is just to make things easier in the future. You can alternatively just type the following to get the current ViewController.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="javascript" style="font-family:monospace;">cy# <span style="color: #009900;">&#91;</span>UIApp.<span style="color: #660066;">keyWindow</span>.<span style="color: #660066;">rootViewController</span>.<span style="color: #660066;">visibleViewController</span><span style="color: #009900;">&#93;</span>
<span style="color: #009900;">&#91;</span><span style="color: #339933;">@</span><span style="color: #3366CC;">"&lt;PatternLockViewController: 0x1d88d4f0"</span><span style="color: #009900;">&#93;</span></pre>
      </td>
    </tr>
  </table>
</div>

**3. Analyze the Lock Screen Controller**

This is the phase where we basically will just start looking for things that we can remove or somehow open the lock screen. With a little luck and common sense things will stick out worth altering.  Below are a few tricks I used to analyze this particular app.

One way to do this is to add a function in Cycript for printing all the methods of a class. This one below was taken from <a href="http://iphonedevwiki.net/index.php/Cycript_Tricks" target="_blank">cycript tricks</a>. You can just copy and paste it into the console and call it with &#8220;PatternLockViewController&#8221;. Reminder that &#8220;PatternLockViewController&#8221; is for this example, yours should be the one you discovered in step two.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="javascript" style="font-family:monospace;">cy# <span style="color: #000066; font-weight: bold;">function</span> printMethods<span style="color: #009900;">&#40;</span>className<span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
cy#    <span style="color: #000066; font-weight: bold;">var</span> count <span style="color: #339933;">=</span> <span style="color: #000066; font-weight: bold;">new</span> <span style="color: #000066; font-weight: bold;">new</span> Type<span style="color: #009900;">&#40;</span><span style="color: #3366CC;">"I"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
cy#    <span style="color: #000066; font-weight: bold;">var</span> methods <span style="color: #339933;">=</span> class_copyMethodList<span style="color: #009900;">&#40;</span>objc_getClass<span style="color: #009900;">&#40;</span>className<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span> count<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
cy#    <span style="color: #000066; font-weight: bold;">var</span> methodsArray <span style="color: #339933;">=</span> <span style="color: #009900;">&#91;</span><span style="color: #009900;">&#93;</span><span style="color: #339933;">;</span>
cy#    <span style="color: #000066; font-weight: bold;">for</span><span style="color: #009900;">&#40;</span><span style="color: #000066; font-weight: bold;">var</span> i <span style="color: #339933;">=</span> <span style="color: #CC0000;"></span><span style="color: #339933;">;</span> i <span style="color: #339933;">&</span>lt<span style="color: #339933;">;</span> <span style="color: #339933;">*</span>count<span style="color: #339933;">;</span> i<span style="color: #339933;">++</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span> cy<span style="color: #339933;">&</span>gt<span style="color: #339933;">;</span>      <span style="color: #000066; font-weight: bold;">var</span> method <span style="color: #339933;">=</span> methods<span style="color: #009900;">&#91;</span>i<span style="color: #009900;">&#93;</span><span style="color: #339933;">;</span>
cy<span style="color: #339933;">&</span>gt<span style="color: #339933;">;</span>      methodsArray.<span style="color: #660066;">push</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:</span>method_getName<span style="color: #009900;">&#40;</span>method<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span> implementation<span style="color: #339933;">:</span>method_getImplementation<span style="color: #009900;">&#40;</span>method<span style="color: #009900;">&#41;</span><span style="color: #009900;">&#125;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
cy<span style="color: #339933;">&</span>gt<span style="color: #339933;">;</span>    <span style="color: #009900;">&#125;</span>
cy<span style="color: #339933;">&</span>gt<span style="color: #339933;">;</span>    free<span style="color: #009900;">&#40;</span>methods<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
cy<span style="color: #339933;">&</span>gt<span style="color: #339933;">;</span>    free<span style="color: #009900;">&#40;</span>count<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
cy<span style="color: #339933;">&</span>gt<span style="color: #339933;">;</span>    <span style="color: #000066; font-weight: bold;">return</span> methodsArray<span style="color: #339933;">;</span>
cy<span style="color: #339933;">&</span>gt<span style="color: #339933;">;</span> <span style="color: #009900;">&#125;</span>
cy# printMethods<span style="color: #009900;">&#40;</span>PatternLockViewController<span style="color: #009900;">&#41;</span>
<span style="color: #009900;">&#91;</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>moreButton<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70aa1<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setMoreButton<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70ac1<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>moreTapped<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x6fd21<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setupView<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x702fd<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>didFinishLockWithKey<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70739<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>lockerView<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70b09<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setupHeaderWithType<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x6fda1<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>requestPassword<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70a49<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setRequestPassword<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70a59<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>drawPatternLockView<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70a15<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setDrawPatternLockView<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70a35<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setLockerView<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70b29<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>tempPassword<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70a69<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>dismissPatternLockViewController<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x7069d<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setTempPassword<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70a79<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setBackgroundView<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x709cd<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>backgroundView<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x709ad<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>viewDidLoad<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x704cd<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>viewDidUnload<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70609<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setHeaderView<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70af5<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>headerView<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70ad5<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>.<span style="color: #660066;">cxx_destruct</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70b3d<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>delegate<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x709e1<span style="color: #009900;">&#125;</span><span style="color: #339933;">,</span><span style="color: #009900;">&#123;</span>selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setDelegate<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70a01<span style="color: #009900;">&#125;</span><span style="color: #009900;">&#93;</span></pre>
      </td>
    </tr>
  </table>
</div>

While the dump doesn&#8217;t display that well in this format below are some interesting methods.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="javascript" style="font-family:monospace;">selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>requestPassword<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70a49
selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>setRequestPassword<span style="color: #339933;">:</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70a59
selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>tempPassword<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x70a69
selector<span style="color: #339933;">:@</span>selector<span style="color: #009900;">&#40;</span>dismissPatternLockViewController<span style="color: #009900;">&#41;</span><span style="color: #339933;">,</span>implementation<span style="color: #339933;">:</span>0x7069d</pre>
      </td>
    </tr>
  </table>
</div>

Another thing to do is dump the messages to ensure you are getting all delegates.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="javascript" style="font-family:monospace;">cy# PatternLockViewController.<span style="color: #660066;">messages</span>
<span style="color: #009900;">&#123;</span>moreButton<span style="color: #339933;">:</span>0x70aa1<span style="color: #339933;">,</span><span style="color: #3366CC;">"setMoreButton:"</span><span style="color: #339933;">:</span>0x70ac1<span style="color: #339933;">,</span><span style="color: #3366CC;">"moreTapped:"</span><span style="color: #339933;">:</span>0x6fd21<span style="color: #339933;">,</span>setupView<span style="color: #339933;">:</span>0x702fd<span style="color: #339933;">,</span><span style="color: #3366CC;">"didFinishLockWithKey:"</span><span style="color: #339933;">:</span>0x70739<span style="color: #339933;">,</span>lockerView<span style="color: #339933;">:</span>0x70b09<span style="color: #339933;">,</span><span style="color: #3366CC;">"setupHeaderWithType:"</span><span style="color: #339933;">:</span>0x6fda1<span style="color: #339933;">,</span>requestPassword<span style="color: #339933;">:</span>0x70a49<span style="color: #339933;">,</span><span style="color: #3366CC;">"setRequestPassword:"</span><span style="color: #339933;">:</span>0x70a59<span style="color: #339933;">,</span>drawPatternLockView<span style="color: #339933;">:</span>0x70a15<span style="color: #339933;">,</span><span style="color: #3366CC;">"setDrawPatternLockView:"</span><span style="color: #339933;">:</span>0x70a35<span style="color: #339933;">,</span><span style="color: #3366CC;">"setLockerView:"</span><span style="color: #339933;">:</span>0x70b29<span style="color: #339933;">,</span>tempPassword<span style="color: #339933;">:</span>0x70a69<span style="color: #339933;">,</span>dismissPatternLockViewController<span style="color: #339933;">:</span>0x7069d<span style="color: #339933;">,</span><span style="color: #3366CC;">"setTempPassword:"</span><span style="color: #339933;">:</span>0x70a79<span style="color: #339933;">,</span><span style="color: #3366CC;">"setBackgroundView:"</span><span style="color: #339933;">:</span>0x709cd<span style="color: #339933;">,</span>backgroundView<span style="color: #339933;">:</span>0x709ad<span style="color: #339933;">,</span>viewDidLoad<span style="color: #339933;">:</span>0x704cd<span style="color: #339933;">,</span>viewDidUnload<span style="color: #339933;">:</span>0x70609<span style="color: #339933;">,</span><span style="color: #3366CC;">"setHeaderView:"</span><span style="color: #339933;">:</span>0x70af5<span style="color: #339933;">,</span>headerView<span style="color: #339933;">:</span>0x70ad5<span style="color: #339933;">,</span><span style="color: #3366CC;">".cxx_destruct"</span><span style="color: #339933;">:</span>0x70b3d<span style="color: #339933;">,</span>delegate<span style="color: #339933;">:</span>0x709e1<span style="color: #339933;">,</span><span style="color: #3366CC;">"setDelegate:"</span><span style="color: #339933;">:</span>0x70a01<span style="color: #339933;">,</span>mf_dataForUICustomization<span style="color: #339933;">:</span>0x3358d741<span style="color: #339933;">,</span>mf_keyPathsMapForUICustomization<span style="color: #339933;">:</span>0x3358d73d<span style="color: #339933;">,</span><span style="color: #3366CC;">"mf_setDataForUI"</span></pre>
      </td>
    </tr>
  </table>
</div>

Some interesting messages to take a look at are.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="javascript" style="font-family:monospace;">dismissPatternLockViewController<span style="color: #339933;">:</span>0x7069d
setRequestPassword<span style="color: #339933;">:</span>0x70a59</pre>
      </td>
    </tr>
  </table>
</div>

Another valuable resource is class-dump-z information. This is the class headers dump that I have done on the application. The example below is an area in the class dump that stood out to me from the data I saw above. I found this just by looking for &#8220;PatternLockViewController&#8221; and &#8220;requestPassword&#8221;. All of which was obtained from the previous actions. If you are unaware of how to get class-dump-z information there are tons of guides on the internet.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="c" style="font-family:monospace;">__attribute__<span style="color: #009900;">&#40;</span><span style="color: #009900;">&#40;</span>visibility<span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"hidden"</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#41;</span>
@interface PatternLockViewController <span style="color: #339933;">:</span> UIViewController  <span style="color: #009900;">&#123;</span>
	UIImageView<span style="color: #339933;">*</span> _backgroundView<span style="color: #339933;">;</span>
	id _delegate<span style="color: #339933;">;</span>
	DrawPatternLockView<span style="color: #339933;">*</span> _drawPatternLockView<span style="color: #339933;">;</span>
	BOOL _requestPassword<span style="color: #339933;">;</span>
	NSString<span style="color: #339933;">*</span> _tempPassword<span style="color: #339933;">;</span>
	UIButton<span style="color: #339933;">*</span> _moreButton<span style="color: #339933;">;</span>
	UIImageView<span style="color: #339933;">*</span> _headerView<span style="color: #339933;">;</span>
	UIImageView<span style="color: #339933;">*</span> _lockerView<span style="color: #339933;">;</span>
<span style="color: #009900;">&#125;</span>
@property<span style="color: #009900;">&#40;</span>assign<span style="color: #339933;">,</span> nonatomic<span style="color: #009900;">&#41;</span> __weak UIImageView<span style="color: #339933;">*</span> lockerView<span style="color: #339933;">;</span>
@property<span style="color: #009900;">&#40;</span>assign<span style="color: #339933;">,</span> nonatomic<span style="color: #009900;">&#41;</span> __weak UIImageView<span style="color: #339933;">*</span> headerView<span style="color: #339933;">;</span>
@property<span style="color: #009900;">&#40;</span>assign<span style="color: #339933;">,</span> nonatomic<span style="color: #009900;">&#41;</span> __weak UIButton<span style="color: #339933;">*</span> moreButton<span style="color: #339933;">;</span>
@property<span style="color: #009900;">&#40;</span>retain<span style="color: #339933;">,</span> nonatomic<span style="color: #009900;">&#41;</span> NSString<span style="color: #339933;">*</span> tempPassword<span style="color: #339933;">;</span>
@property<span style="color: #009900;">&#40;</span>assign<span style="color: #339933;">,</span> nonatomic<span style="color: #009900;">&#41;</span> __weak UIImageView<span style="color: #339933;">*</span> backgroundView<span style="color: #339933;">;</span>
@property<span style="color: #009900;">&#40;</span>assign<span style="color: #339933;">,</span> nonatomic<span style="color: #009900;">&#41;</span> BOOL requestPassword<span style="color: #339933;">;</span>
@property<span style="color: #009900;">&#40;</span>assign<span style="color: #339933;">,</span> nonatomic<span style="color: #009900;">&#41;</span> __weak id delegate<span style="color: #339933;">;</span>
@property<span style="color: #009900;">&#40;</span>assign<span style="color: #339933;">,</span> nonatomic<span style="color: #009900;">&#41;</span> __weak DrawPatternLockView<span style="color: #339933;">*</span> drawPatternLockView<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>.<span style="color: #202020;">cxx_destruct</span><span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>didFinishLockWithKey<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>key<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>dismissPatternLockViewController<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>viewDidUnload<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>viewDidLoad<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>setupView<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>setupHeaderWithType<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">int</span><span style="color: #009900;">&#41;</span>type<span style="color: #339933;">;</span>
<span style="color: #339933;">-</span><span style="color: #009900;">&#40;</span><span style="color: #993333;">void</span><span style="color: #009900;">&#41;</span>moreTapped<span style="color: #339933;">:</span><span style="color: #009900;">&#40;</span>id<span style="color: #009900;">&#41;</span>tapped<span style="color: #339933;">;</span>
@end</pre>
      </td>
    </tr>
  </table>
</div>

&nbsp;

**4. Inject the application.**

With all the information we gathered above there is a couple of things we can do here and probably more. Below are just two methods that get us access to the application. One is setting a variable and the other is setting your View Controller.

**Setting Variable:**

We will want to set the &#8220;_BOOL _requestPassword_&#8221; variable. Being a BOOLEAN we can easily change this variable to true in cycript like below. Notice calling the controller with the variable requestPassword and returning true.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="javascript" style="font-family:monospace;">cy# PatternLockViewController.<span style="color: #660066;">messages</span><span style="color: #009900;">&#91;</span><span style="color: #3366CC;">'requestPassword'</span><span style="color: #009900;">&#93;</span> <span style="color: #339933;">=</span> <span style="color: #000066; font-weight: bold;">function</span><span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span><span style="color: #000066; font-weight: bold;">return</span> <span style="color: #003366; font-weight: bold;">true</span><span style="color: #339933;">;</span><span style="color: #009900;">&#125;</span>
<span style="color: #000066; font-weight: bold;">function</span> <span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span><span style="color: #000066; font-weight: bold;">return</span> <span style="color: #003366; font-weight: bold;">true</span><span style="color: #339933;">;</span><span style="color: #009900;">&#125;</span></pre>
      </td>
    </tr>
  </table>
</div>

In this particular example it called for me to set the master password again. I was then able to set a new password and get into the &#8220;secure&#8221; container.

**Set View Controller:**

Secondly we can just call &#8220;_dismissPatternLockViewController_&#8221; as the visible controller. This will not change the master password like above but instead just get rid of the password screen thus allowing access to all the data.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="javascript" style="font-family:monospace;">cy# <span style="color: #009900;">&#91;</span>UIApp.<span style="color: #660066;">keyWindow</span>.<span style="color: #660066;">rootViewController</span>.<span style="color: #660066;">visibleViewController</span> dismissPatternLockViewController<span style="color: #009900;">&#93;</span></pre>
      </td>
    </tr>
  </table>
</div>

Hopefully this was a decent enough primer to get you started with Cycript. If anyone else has some standard things they do in cyript I would love to hear them.

**References:**

  * <a href="http://www.slideshare.net/Shakacon/andreas-kurtz" target="_blank">http://www.slideshare.net/Shakacon/andreas-kurtz</a>
  * <a href="http://www.slideshare.net/null0x00/breaking-ios-apps-using-cycript" target="_blank">http://www.slideshare.net/null0x00/breaking-ios-apps-using-cycript</a>
  * <a href="http://iphonedevwiki.net/index.php/Cycript_Tricks" target="_blank">http://iphonedevwiki.net/index.php/Cycript_Tricks</a>
  * <a href="http://www.cycript.org" target="_blank">http://www.cycript.org</a>
  * <a href="http://iphonedevwiki.net/index.php/Cycript" target="_blank">http://iphonedevwiki.net/index.php/Cycript</a>
  * <a href="http://highaltitudehacks.com/2013/07/25/ios-application-security-part-8-method-swizzling-using-cycript/" target="_blank">http://highaltitudehacks.com/2013/07/25/ios-application-security-part-8-method-swizzling-using-cycript/</a>
  * <a href="http://iphonedevwiki.net/index.php/MobileSubstrate" target="_blank">http://iphonedevwiki.net/index.php/MobileSubstrate</a>