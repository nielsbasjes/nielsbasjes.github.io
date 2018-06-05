---
layout: post
title: Making sense of the user agent string
date: '2017-03-30T12:00:00+01:00'
author: Niels Basjes
tags:
- WebAnalytics
- Useragent
---

This is a repost of the blog I wrote for the bol.com techlab site: <a href="https://techlab.bol.com/making-sense-user-agent-string/">https://techlab.bol.com/making-sense-user-agent-string/</a>


<p>Ever since I’ve started working for a WebAnalytics company in 2005 I’ve been working on problems related to making sense of web data. One of the most difficult elements in this type of analysis is making sense of the user agent.</p>
<p>Very often the raw web data I work with is stored in Apache HTTPD access log files that have been compressed using gzip. Because I’m a software developer at heart, it will not surprise you that over the years I’ve written several tools to work more effectively with these types of files. Think of easily extracting the various elements of log lines (<a style="background-color: #ffffff;" href="https://github.com/nielsbasjes/logparser">Apache HTTPD &amp; NGINX access log parser</a>) and speeding up the analysis of multi-GiB text files in an Apache Hadoop cluster (<a style="background-color: #ffffff;" href="https://github.com/nielsbasjes/splittablegzip">Hadoop Splittable Gzip Codec</a>). This blog post is about the tool <a href="https://github.com/nielsbasjes/yauaa">Yauaa</a> (Yet Another User Agent Analyzer) that I created to make it easier to make sense of the user agent.</p>
<p>In his post on <a href="/better-device-profiling/">Better device profiling</a> my colleague <a href="https://techlab.bol.com/author/hvanbuuren/">Hans van Buuren</a> describes how bol.com uses <a href="https://github.com/nielsbasjes/yauaa">Yauaa</a>. My post digs deeper into the background and the details on the how and why of this system.</p>
<h1 id="the-user-agent-header">The User-Agent request header</h1>
<p>When a web browser does an HTTP request to a website, one of the headers in such a request is the User-Agent header as described in <a href="https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.43">RFC 2616</a>:</p>
<p style="padding-left: 30px;"><code>The User-Agent request-header field contains information about the user agent originating the request. This is for statistical purposes, the tracing of protocol violations, and automated recognition of user agents for the sake of tailoring responses to avoid particular user agent limitations.</code></p>
<p>An example of a user agent is this:</p>
<p style="padding-left: 30px;"><code>Mozilla/5.0 (Android 4.4; Tablet; rv:41.0) Gecko/41.0 Firefox/41.0</code></p>
<p>It clearly indicates the type and version of the operating system, the device type and the browser name and version. Nice!</p>
<h1 id="serving-optimal-web-content-to-the-visitor">Serving optimal web content to the visitor</h1>
<p>In the early days of web browsers the differences in supported features were huge. Websites that tried to give users the best possible usability resorted to using features supported in some web browsers only. As a consequence they needed to include code that effectively enabled or disabled features to make the websites ‘look better’ if the browser supported it.</p>
<p>Because client-side code support (like javascript) was limited, many websites chose to use very simple server-side code to make the choice between sending back the ‘basic’ or the ‘enhanced usability’ version of their website.</p>
<p>They essentially did something like this in their web server:</p>
<p style="padding-left: 30px;"><code>if (useragent.contains("Mozilla/")) {<br />
// Use the Netscape feature we need.<br />
} else {<br />
// Fallback to the simpler site<br />
}</code></p>
<p>Some sites improved on this by actually checking if the version is high enough.</p>
<p>After a while the competing web browser caught up and released a new version which implemented the features needed to show the ‘enhanced usability’ versions of that website. Unfortunately, the website would still be served in its basic version as their user agent couldn&#8217;t match the pattern that would trigger the better version. The solution many have chosen was simple and effective: add extra information to the User-Agent header to indicate the browsers you are compatible with.</p>
<h1 id="lies-damned-lies-and-useragents">Lies, damned lies and user agents</h1>
<p>At first glance it makes a lot of sense to add this extra compatibility information. The reality, however, is that this is the reason the current user agents are such a mess. Let me illustrate with this real user agent pulled from our production access log files of February 2017:</p>
<p style="padding-left: 30px;"><code>Mozilla/5.0 (Windows Phone 10.0; Android 6.0.1; Microsoft; Lumia 650) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/51.0.2704.79 Mobile Safari/537.36 Edge/14.14393</code></p>
<p>Let’s examine the various parts we find and assess them:</p>
<table>
<tbody>
<tr>
<td>Mozilla/5.0</td>
<td><strong>No</strong>, this is not Mozilla Netscape 5.0</td>
</tr>
<tr>
<td>Windows Phone 10.0</td>
<td><strong>Yes</strong>, a Windows Phone device</td>
</tr>
<tr>
<td>Android 6.0.1</td>
<td><strong>No</strong>, not Android</td>
</tr>
<tr>
<td>Microsoft; Lumia 650</td>
<td><strong>Yes</strong>, a Microsoft (Nokia) Lumia 650 device</td>
</tr>
<tr>
<td>AppleWebKit/537.36</td>
<td><strong>No</strong>, not AppleWebKit</td>
</tr>
<tr>
<td>KHTML</td>
<td><strong>No</strong>, not KHTML (Konqueror)</td>
</tr>
<tr>
<td>like Gecko</td>
<td><strong>No</strong>, not Gecko based</td>
</tr>
<tr>
<td>Chrome/51.0.2704.79</td>
<td><strong>No</strong>, not Chrome</td>
</tr>
<tr>
<td>Mobile Safari/537.36</td>
<td><strong>No</strong>, not Mobile Safari &#8230;</td>
</tr>
<tr>
<td>Edge/14.14393</td>
<td><strong>Yes</strong>, this is the Microsoft Edge browser</td>
</tr>
</tbody>
</table>
<p>So out of the 10 elements we see here, there are 7 lies to indicate the tools they&#8217;re compatible with. Now, don’t blame Microsoft this time; almost all of the browsers do this.</p>
<h1 id="but-what-do-i-really-need-to-know">But what do I really need to know?</h1>
<p>Before we start digging into the user agents, we first need to answer the fundamental question: what do we really need to know and what do we intend to do with it? In this context I have found two fundamental situations:</p>
<ol>
<li>High level:
<ul>
<li>What class of device is this?</li>
<li>What operating system, browser, etc. is this? This is useful for analytics purposes and for situations (like we have at bol.com) where we have a ‘full-size’ site and a ‘light-weight’ mobile site.</li>
</ul>
</li>
<li>Detailed:
<ul>
<li>What screen resolution does this device have and does this browser support the latest javascript/css/… feature?</li>
</ul>
</li>
</ol>
<p>There are projects like <a href="http://www.browserscope.org">BrowserScope</a> and <a href="http://attic.apache.org/projects/devicemap.html">Apache DeviceMap</a> that try (tried) to maintain a database of supported detailed features. As you can understand this is a lot of work to maintain because of the high change rate of new devices and browsers. In general these databases will always be ‘too old’ for some applications.</p>
<p>I have found that if you need to know the detailed information the only <em>workable</em> way of doing this is by using some scripting (usually javascript) to determine the support for these features in the browser itself. There are tools available that support this idea (like <a href="https://modernizr.com/">Modernizr</a>).</p>
<p>In my work I only have the user agent string in some kind of log file and I only need the high-level information. At that point in time, running any JavaScript is no longer possible and it is also not needed for my use case.</p>
<h1 id="analyzing-the-user-agent-string">Analyzing the user agent string</h1>
<p>Back in 2005 I wrote my first user agent analyzer for reporting purposes. It used sequences of rules in a (proprietary) pattern matching language and the first rule that matched was used for the output. This worked quite well for the attributes I needed (things like Operating System and Browser).</p>
<p>Yet after a while I found that the variations in the user agent strings was so large it became a maintenance nightmare. The main issue is that the ordering of some elements varied and putting N! rules for the required combination of N explicit elements quickly becomes a problem.</p>
<p>In 2008 I started working at bol.com and after a while I found <a href="https://github.com/ua-parser">UA-Parser</a> for which I wrote the <a href="https://github.com/ua-parser/uap-pig">Apache Pig UDF</a>. UA-Parser effectively does the same as what I wrote back in 2005. The main difference is that it uses regular expressions, making the set of rules easily portable to many programming languages. The downside is that it uses regular expressions which makes it a bit harder to maintain.</p>
<p>The systems that work based on an ordered list of rules suffer from a fundamental performance problem. The exceptions to the normal (common) pattern are at the start of the list that is checked, the ‘clean’ situations at the bottom. So in practice, the most common situations are at the bottom so that in most cases a very large number of rules needs to be checked.</p>
<h1 id="can-it-be-done-differently">Can it be done differently?</h1>
<p>Back in 2011 I started thinking: can this be done differently? Can I actually do this without sequential lists of rules? Can it be made so that anyone can add new rules?</p>
<p>I quickly realized that perhaps the user agent could be parsed using a real parser. But … there is only a very unclear language specification and many browsers don&#8217;t follow this specification at all. So this parser must be able to handle some parse errors without failing too hard.</p>
<p>Worst of all: this is also an example I want to be able to properly classify:</p>
<p class="" style="padding-left: 30px;"><code>Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:36.0) Gecko/20100101 Firefox/36.0')waitfor delay'0:0:20'--</code></p>
<p>Yes, this is an SQL injection attack using the user agent. So no “Firefox” or “Linux” but “Hacker”.</p>
<p>At first I did some experiments with Antlr3 to see if I could create a parser. I wanted to transform the user agent string into a predictable tree structure on which I could turn a pattern analyzer loose. Using Antlr3 I couldn’t get it do do what I wanted, probably caused by my own limited knowledge at the time.</p>
<p>A few years later (2015) I tried it again with <a href="http://www.antlr.org/">Antlr4</a>. This finally succeeded in early 2016 (I&#8217;d gotten the <a href="https://www.bol.com/nl/p/the-definitive-antlr-4-reference/9200000005546647/">Antlr4 book</a> for Christmas). One of the main reasons it succeeded this time (other than reading the manual) is that Antlr4 is capable of creating a ‘mostly right’ tree even in cases where the input doesn&#8217;t fully follow the described pattern. So some of the ‘catching errors’ is done by Antlr4.</p>
<h1 id="so-now-i-have-a-tree-lets-match-some-patterns.">So now I have a tree, let’s match some patterns</h1>
<p>Having seen the high rate of change in the available devices and browsers I chose to limit the number of lookup tables and browser/device specific rules. So the main focus of the rules I did write, is on ‘extracting’ information from the user agent, not ‘looking up information’.</p>
<p>Also, the pattern matching system has been designed to handle a very large number of rules as efficiently as possible.</p>
<p>One of the key differences with other systems is that a tree structure is available for the user agent. It becomes possible to look for a pattern and walk through all the elements of the tree to find the value that needs to be extracted. At startup each rule set (called Matcher) has registered itself in a HashMap indicating which nodes of the tree are of interest.</p>
<p>Essentially Yauaa now works as follows:</p>
<ol>
<li>Parse the user agent</li>
<li>Walk through all the elements of the tree
<ul>
<li>Fire each tree element against the rules that have indicated interest in that element.</li>
</ul>
</li>
<li>Ask each rule set if it has enough input
<ul>
<li>If enough input evaluate the rules</li>
<li>If all rules success return the result</li>
</ul>
</li>
<li>Aggregate all returned results into a final answer</li>
</ol>
<p>For a very common &#8216;Chrome on Windows&#8217; user agent we can now extract all relevant information from over 1800 rule sets in about 0.3 milliseconds. It can run even faster if only specific fields are requested.</p>
<h1 id="and-then">And then …</h1>
<p>in the summer of 2016 I asked some of my colleagues what they thought of my hobby project. It happened to be exactly what they were looking for because the Apache DeviceMap project (on which some of the tools were running) was no longer being updated. My colleague Hans van Buuren has already written in <a href="https://techlab.bol.com/better-device-profiling/">his blog</a> about that transition.</p>
<h1 id="looking-back">Looking back</h1>
<p>I started this project a long time ago as an experiment to see &#8216;if it can be done differently&#8217;.</p>
<p>The answer today is: yes, it surely can and it is very efficient too.</p>
<p>I made this project open source under the Apache 2.0 licence. So if this solves a problem for you then go to <a href="https://github.com/nielsbasjes/yauaa">https://github.com/nielsbasjes/yauaa</a> and read how you can use and run it.</p>
