---
id: 64
title: Testing iOS 7 Apps Without Root
date: 2013-12-12T17:32:48+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=64
permalink: /?p=64
categories:
  - mobile
tags:
  - Burp
  - iOS
  - MITM
  - mobile
---
At the time of writing this there is no public jailbreak for iOS 7.0.4, but the requests for testing applications within iOS still keep coming in. While the best approach to this request is to test the application on an[<img class="alignright  wp-image-67" alt="No iOS7 jailbreak" src="http://www.infointox.net/wp-content/uploads/2013/12/iOS7-jailbreak-300x202.png" width="300" height="202" srcset="http://www.infointox.net/wp-content/uploads/2013/12/iOS7-jailbreak-300x202.png 300w, http://www.infointox.net/wp-content/uploads/2013/12/iOS7-jailbreak.png 480w" sizes="(max-width: 300px) 100vw, 300px" />](http://www.infointox.net/wp-content/uploads/2013/12/iOS7-jailbreak.png) older device that you have root on and analyze any difference iOS 7 may introduce but some customers just don’t want to take that approach. Finding myself in many of these situations I wanted to document the approach I took to testing iOS applications without root. Without root you are extremely limited to what you can or cannot do to access the application. Below I break it down into basically three different steps **Console Messages**, **Browse Filesystem**, and **Intercept Communication**.

&nbsp;

**Console Messages:**

While very trivial it has provided me with some results. Just physically attach the phone to your Mac with Xcode open and monitor the console while you browse around the app on the phone. Since these messages are written to the console in an unencrypted fashion nothing should be sensitive here. You would be amazed how many apps are logging the user & password within a URL string while it authenticates back to the server. This type of information has lead me to some pretty nice attacks.

&nbsp;

<p style="text-align: center;">
  <a href="http://www.infointox.net/wp-content/uploads/2013/12/Console-Log.png"><img class=" wp-image-66 aligncenter" alt="Console Log" src="http://www.infointox.net/wp-content/uploads/2013/12/Console-Log-1024x248.png" width="819" height="198" srcset="http://www.infointox.net/wp-content/uploads/2013/12/Console-Log-1024x248.png 1024w, http://www.infointox.net/wp-content/uploads/2013/12/Console-Log-300x72.png 300w, http://www.infointox.net/wp-content/uploads/2013/12/Console-Log.png 1340w" sizes="(max-width: 819px) 100vw, 819px" /></a>
</p>

&nbsp;

**Browse Filesystem:**

Another very trivial thing to do. While the phone is connected use your favorite file system browser to see if anything is exposed. If everything was done correctly within the application nothing of value should be able to be accessed by this method. Again you will be amazed of what some applications allow you to have access to simply by connecting via USB. I use <a href="http://www.digidna.net/diskaid" target="_blank">DiskAid</a> below and many people also like <a href="http://www.macroplant.com/iexplorer/" target="_blank">iExplorer</a>. You could also forensically recover the file system and attempt to recover some older versions of the files you find.

<p style="text-align: center;">
    <a href="http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-11-22-at-2.20.09-PM.png"><img class="aligncenter  wp-image-73" alt="browse" src="http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-11-22-at-2.20.09-PM-1024x440.png" width="819" height="352" srcset="http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-11-22-at-2.20.09-PM-1024x440.png 1024w, http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-11-22-at-2.20.09-PM-300x128.png 300w, http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-11-22-at-2.20.09-PM.png 1063w" sizes="(max-width: 819px) 100vw, 819px" /></a> <a href="http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-11-22-at-2.20.09-PM.png"><br /> </a>
</p>

&nbsp;

**Intercept Communication(MITM):**

Something that can usually be pretty fruitful is intercepting the application traffic and begin to test the client/server traffic just as you would any other web application. From my experience many apps are not expecting you to get in the middle so some basic security measures may not have been implemented. This approach definitely lets you see under the hood a bit more but from here on it essentially ends up being just like any other web based penetration test.  I won’t go into how to web test but I will show you how to setup Burp to get in the middle of your app.  The best part about this is once you have it all setup you don’t need to go through this whole process again. Whenever I want to test an app now I just enable the proxy on the phone and fire up Burp.

&nbsp;

**1. Setup Burp & Export Burp Certificate**

On your machine setup Burp to listen on all interfaces like below. We also need to export the Burp CA certificate so we can import it into the iPhone. Now if DNS pinning is used you can stop here because this will not work.

<p style="text-align: center;">
  <a href="http://www.infointox.net/wp-content/uploads/2013/12/burp-proxy-setup.png"><img class="aligncenter  wp-image-69" alt="burp proxy setup" src="http://www.infointox.net/wp-content/uploads/2013/12/burp-proxy-setup.png" width="802" height="342" srcset="http://www.infointox.net/wp-content/uploads/2013/12/burp-proxy-setup.png 1003w, http://www.infointox.net/wp-content/uploads/2013/12/burp-proxy-setup-300x128.png 300w" sizes="(max-width: 802px) 100vw, 802px" /></a>
</p>

&nbsp;

Export Certificate in DER format and save it to a working directory that is easy to get at.

<p style="text-align: center;">
   <a href="http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-12-12-at-11.52.14-AM.png"><img class="aligncenter  wp-image-74" alt="DER CA Cert Export" src="http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-12-12-at-11.52.14-AM.png" width="478" height="350" srcset="http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-12-12-at-11.52.14-AM.png 598w, http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-12-12-at-11.52.14-AM-300x219.png 300w" sizes="(max-width: 478px) 100vw, 478px" /></a>
</p>

&nbsp;

**2. Setup your iPhone **

First install the Burp certificate that was just exported. The easiest way is to start a simple python HTTP server and browse to that on your iPhone.  To start the HTTP server go to the certificate location you just exported and run the following command.

**python -m SimpleHTTPServer**

 ****Now you can browse to your machine via the iPhone and simply click on the Burp CA certificate to start the install process. You should see similar screens like below. _Note the default port is 8000. _

&nbsp;

[<img class="aligncenter size-full wp-image-65" alt="ios7 browse" src="http://www.infointox.net/wp-content/uploads/2013/12/ios7-browse.png" width="637" height="230" srcset="http://www.infointox.net/wp-content/uploads/2013/12/ios7-browse.png 637w, http://www.infointox.net/wp-content/uploads/2013/12/ios7-browse-300x108.png 300w" sizes="(max-width: 637px) 100vw, 637px" />](http://www.infointox.net/wp-content/uploads/2013/12/ios7-browse.png)

[<img class=" wp-image-71 alignleft" alt="ios7 install cert1" src="http://www.infointox.net/wp-content/uploads/2013/12/ios7-install-cert1-296x300.png" width="237" height="240" srcset="http://www.infointox.net/wp-content/uploads/2013/12/ios7-install-cert1-296x300.png 296w, http://www.infointox.net/wp-content/uploads/2013/12/ios7-install-cert1.png 636w" sizes="(max-width: 237px) 100vw, 237px" />](http://www.infointox.net/wp-content/uploads/2013/12/ios7-install-cert1.png)

<img class="alignright  wp-image-75" alt="" src="http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-12-12-at-12.48.41-PM-300x300.png" width="240" height="240" srcset="http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-12-12-at-12.48.41-PM-300x300.png 300w, http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-12-12-at-12.48.41-PM-150x150.png 150w, http://www.infointox.net/wp-content/uploads/2013/12/Screen-Shot-2013-12-12-at-12.48.41-PM.png 624w" sizes="(max-width: 240px) 100vw, 240px" />

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

Now that we have the Burp CA installed we just need to set the iPhone proxy to point to our machine which has the Burp proxy listening. This can be done by going to the following.

**Settings > Wi-Fi > SSID > HTTP PROXY**

&nbsp;

<p style="text-align: center;">
  <a href="http://www.infointox.net/wp-content/uploads/2013/12/ios7-proxy.png"><img class="aligncenter  wp-image-72" alt="ios7 proxy" src="http://www.infointox.net/wp-content/uploads/2013/12/ios7-proxy.png" width="305" height="223" srcset="http://www.infointox.net/wp-content/uploads/2013/12/ios7-proxy.png 635w, http://www.infointox.net/wp-content/uploads/2013/12/ios7-proxy-300x219.png 300w" sizes="(max-width: 305px) 100vw, 305px" /></a>
</p>

&nbsp;

**3. Start Testing!**

&nbsp;

<p style="text-align: center;">
  <a href="http://www.infointox.net/wp-content/uploads/2013/12/start-testing.png"><img class="aligncenter  wp-image-76" alt="start testing" src="http://www.infointox.net/wp-content/uploads/2013/12/start-testing.png" width="700" height="251" srcset="http://www.infointox.net/wp-content/uploads/2013/12/start-testing.png 875w, http://www.infointox.net/wp-content/uploads/2013/12/start-testing-300x107.png 300w" sizes="(max-width: 700px) 100vw, 700px" /></a>
</p>

&nbsp;

_Note: Make sure you install your target app before you MITM.  The app store doesn’t like it so much when you do that._

So that is pretty much what you are limited to if you do not have a jailbreak handy or your customer insists you test on iOS 7(This is pre public jailbreak) I personally would try and convince them otherwise. Now depending on the app you may be able to use it a bit and see various misconfigurations(back up to iCloud, screenshots, etc) but that will vary app to app. If anyone else has ran into similar situations I would love to hear if you had any other methods of attacking the app without root on the phone.