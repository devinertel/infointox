---
id: 17
title: Monitor Web w/ googs.pl
date: 2006-05-11T19:01:00+00:00
author: Devin Ertel
layout: post
guid: http://www.infointox.net/?p=17
permalink: /?p=17
blogger_blog:
  - informationintoxication.blogspot.com
blogger_author:
  - Devinhttp://www.blogger.com/profile/15793900111446118223noreply@blogger.com
blogger_permalink:
  - /2006/05/monitor-web-w-googspl_11.html
categories:
  - Code
tags:
  - Perl
---
I wanted a way to monitor the web for certain terms(i.e. leaked info on a company). For example, being able to have an arrary of search terms and operators to query aganist, and then email me a nice little html report. This is the reason for googs.pl.

I used google API to do the querys. I also (although not sure how well it works) append the google operator **daterange:** which needs the julian date, thus hoping to only return new results that day and only email me if it does find new ones. This way I don&#8217;t have to look at old stuff all the time or get tons of email. You can comment that feature out if you dont want it. To figure the date I used the perl module Cal::Date which I posted a link below. Then I just set it up in a cronjob to run everyday.

<div class="wp_syntax">
  <table>
    <tr>
      <td class="code">
        <pre class="perl" style="font-family:monospace;"><span style="color: #666666; font-style: italic;"># Devin Ertel</span>
<span style="color: #666666; font-style: italic;"># googs.pl</span>
<span style="color: #666666; font-style: italic;"># </span>
<span style="color: #666666; font-style: italic;">#!/usr/bin/perl</span>
&nbsp;
<span style="color: #000000; font-weight: bold;">use</span> strict<span style="color: #339933;">;</span>     
<span style="color: #000000; font-weight: bold;">use</span> SOAP<span style="color: #339933;">::</span><span style="color: #006600;">Lite</span><span style="color: #339933;">;</span>
<span style="color: #000000; font-weight: bold;">use</span> MIME<span style="color: #339933;">::</span><span style="color: #006600;">Lite</span><span style="color: #339933;">;</span>
<span style="color: #000000; font-weight: bold;">use</span> Net<span style="color: #339933;">::</span><span style="color: #006600;">SMTP</span><span style="color: #339933;">;</span>
<span style="color: #000000; font-weight: bold;">use</span> Cal<span style="color: #339933;">::</span><span style="color: #006600;">Date</span> <span style="color: #000066;">qw</span><span style="color: #009900;">&#40;</span>DJM MJD today<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#Get Todays Date</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$date</span> <span style="color: #339933;">=</span> today<span style="color: #009900;">&#40;</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#convert to julian</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$jul_today</span><span style="color: #339933;">=</span> DJM<span style="color: #009900;">&#40;</span><span style="color: #0000ff;">$date</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#Put Your Google API Key Here</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$google_key</span><span style="color: #339933;">=</span><span style="color: #ff0000;">'your_google_key_here'</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#Google WSDL File Location</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$google_wsdl</span> <span style="color: #339933;">=</span> <span style="color: #ff0000;">"./GoogleSearch.wsdl"</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#Put querys here, escape any "'s with \" </span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$query</span><span style="color: #339933;">;</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">@query</span> <span style="color: #339933;">=</span> <span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"company + hacking"</span><span style="color: #339933;">,</span>
	     <span style="color: #ff0000;">"allintext:company + hacking"</span><span style="color: #339933;">,</span>
             <span style="color: #ff0000;">"your querys"</span>
	     <span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
&nbsp;
<span style="color: #666666; font-style: italic;">#assign current julian date to query</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$goog_daterange</span> <span style="color: #339933;">=</span> <span style="color: #ff0000;">" + daterange:"</span><span style="color: #339933;">.</span><span style="color: #0000ff;">$jul_today</span><span style="color: #339933;">.</span><span style="color: #ff0000;">"-"</span><span style="color: #339933;">.</span><span style="color: #0000ff;">$jul_today</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#SOAP::Lite instance with GoogleSearch.wsdl.</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$google_soap</span> <span style="color: #339933;">=</span> SOAP<span style="color: #339933;">::</span><span style="color: #006600;">Lite</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span>service<span style="color: #009900;">&#40;</span><span style="color: #ff0000;">"file:$google_wsdl"</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
&nbsp;
<span style="color: #666666; font-style: italic;">#Set Up Mail Vars</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$faddy</span> <span style="color: #339933;">=</span> <span style="color: #ff0000;">'from_address@blah.com'</span><span style="color: #339933;">;</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$taddy</span> <span style="color: #339933;">=</span> <span style="color: #ff0000;">'to_address@blah.com'</span><span style="color: #339933;">;</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$mail_host</span> <span style="color: #339933;">=</span> <span style="color: #ff0000;">'your_mail_host'</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$subject</span> <span style="color: #339933;">=</span> <span style="color: #ff0000;">"New Information Posted!"</span><span style="color: #339933;">;</span>
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$msg_body</span> <span style="color: #339933;">=</span><span style="color: #ff0000;">""</span><span style="color: #339933;">;</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#Its Google Time</span>
&nbsp;
<span style="color: #666666; font-style: italic;">#Loop Through Array of Querys</span>
<span style="color: #b1b100;">foreach</span> <span style="color: #0000ff;">$query</span> <span style="color: #009900;">&#40;</span><span style="color: #0000ff;">@query</span><span style="color: #009900;">&#41;</span><span style="color: #009900;">&#123;</span>
&nbsp;
	<span style="color: #666666; font-style: italic;">#add daterange: operator to curren query</span>
	<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$query_date</span><span style="color: #339933;">=</span><span style="color: #0000ff;">$query</span><span style="color: #339933;">.</span><span style="color: #0000ff;">$goog_daterange</span><span style="color: #339933;">;</span>
&nbsp;
	<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$results</span> <span style="color: #339933;">=</span> <span style="color: #0000ff;">$google_soap</span> <span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> 
    		doGoogleSearch<span style="color: #009900;">&#40;</span>
      			<span style="color: #0000ff;">$google_key</span><span style="color: #339933;">,</span> <span style="color: #0000ff;">$query_date</span> <span style="color: #339933;">,</span> <span style="color: #cc66cc;"></span><span style="color: #339933;">,</span> <span style="color: #cc66cc;">10</span><span style="color: #339933;">,</span> <span style="color: #ff0000;">"false"</span><span style="color: #339933;">,</span> <span style="color: #ff0000;">""</span><span style="color: #339933;">,</span>  <span style="color: #ff0000;">"false"</span><span style="color: #339933;">,</span>
      			<span style="color: #ff0000;">""</span><span style="color: #339933;">,</span> <span style="color: #ff0000;">"latin1"</span><span style="color: #339933;">,</span> <span style="color: #ff0000;">"latin1"</span>
    		<span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
&nbsp;
	<span style="color: #666666; font-style: italic;"># Exit On No Results</span>
	<span style="color: #339933;">@</span><span style="color: #009900;">&#123;</span><span style="color: #0000ff;">$results</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #009900;">&#123;</span>resultElements<span style="color: #009900;">&#125;</span><span style="color: #009900;">&#125;</span> <span style="color: #b1b100;">or</span> <span style="color: #000066;">exit</span><span style="color: #339933;">;</span>
&nbsp;
	<span style="color: #666666; font-style: italic;"># Loop Results and Output to HTML</span>
	<span style="color: #b1b100;">foreach</span> <span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$result</span> <span style="color: #009900;">&#40;</span><span style="color: #339933;">@</span><span style="color: #009900;">&#123;</span><span style="color: #0000ff;">$results</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #009900;">&#123;</span>resultElements<span style="color: #009900;">&#125;</span><span style="color: #009900;">&#125;</span><span style="color: #009900;">&#41;</span> <span style="color: #009900;">&#123;</span>
&nbsp;
        <span style="color: #666666; font-style: italic;">#had to take brackets out for this post for the html breaks and lines</span>
	<span style="color: #0000ff;">$msg_body</span> <span style="color: #339933;">.=</span> <span style="color: #ff0000;">"br"</span><span style="color: #339933;">.</span>
  		      <span style="color: #0000ff;">$result</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #009900;">&#123;</span><span style="color: #ff0000;">'title'</span><span style="color: #009900;">&#125;</span><span style="color: #339933;">.</span><span style="color: #ff0000;">"br"</span><span style="color: #339933;">.</span>
  		      <span style="color: #ff0000;">"a href="</span><span style="color: #339933;">.</span><span style="color: #0000ff;">$result</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #009900;">&#123;</span>URL<span style="color: #009900;">&#125;</span><span style="color: #339933;">.</span><span style="color: #ff0000;">"&gt;"</span><span style="color: #339933;">.</span><span style="color: #0000ff;">$result</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #009900;">&#123;</span>URL<span style="color: #009900;">&#125;</span><span style="color: #339933;">.</span><span style="color: #ff0000;">"/a br"</span><span style="color: #339933;">.</span>
  		      <span style="color: #0000ff;">$result</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #009900;">&#123;</span>snippet<span style="color: #009900;">&#125;</span><span style="color: #339933;">.</span>
		      <span style="color: #ff0000;">"
hr"</span><span style="color: #339933;">;</span>
&nbsp;
	<span style="color: #009900;">&#125;</span>
<span style="color: #009900;">&#125;</span>
<span style="color: #666666; font-style: italic;">#Setup Message</span>
&nbsp;
<span style="color: #b1b100;">my</span> <span style="color: #0000ff;">$msg</span><span style="color: #339933;">=</span>MIME<span style="color: #339933;">::</span><span style="color: #006600;">Lite</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #000000; font-weight: bold;">new</span> <span style="color: #009900;">&#40;</span>
        From <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #0000ff;">$faddy</span><span style="color: #339933;">,</span>
        To <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #0000ff;">$taddy</span><span style="color: #339933;">,</span>
        Subject <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #0000ff;">$subject</span><span style="color: #339933;">,</span>
	Type <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #ff0000;">'TEXT/HTML'</span><span style="color: #339933;">,</span>
	Encoding <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #ff0000;">'quoted-printable'</span><span style="color: #339933;">,</span>
	Data <span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span> <span style="color: #0000ff;">$msg_body</span><span style="color: #339933;">,</span>
<span style="color: #009900;">&#41;</span>       <span style="color: #b1b100;">or</span> <span style="color: #000066;">die</span> <span style="color: #ff0000;">"Could Not Create Msg: $!<span style="color: #000099; font-weight: bold;">\n</span>"</span><span style="color: #339933;">;</span>
&nbsp;
&nbsp;
<span style="color: #666666; font-style: italic;">#Send Message</span>
MIME<span style="color: #339933;">::</span><span style="color: #006600;">Lite</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #000066;">send</span><span style="color: #009900;">&#40;</span><span style="color: #ff0000;">'smtp'</span><span style="color: #339933;">,</span> <span style="color: #0000ff;">$mail_host</span><span style="color: #339933;">,</span> Timeout<span style="color: #339933;">=&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #cc66cc;">60</span><span style="color: #009900;">&#41;</span><span style="color: #339933;">;</span>
<span style="color: #0000ff;">$msg</span><span style="color: #339933;">-&</span><span style="color: #b1b100;">gt</span><span style="color: #339933;">;</span><span style="color: #000066;">send</span><span style="color: #339933;">;</span></pre>
      </td>
    </tr>
  </table>
</div>

**References:**
  
<http://freshmeat.net/projects/caldate/>
  
<http://www.google.com/apis/>
  
<http://search.cpan.org/~yves/MIME-Lite-3.01/lib/MIME/Lite.pm>