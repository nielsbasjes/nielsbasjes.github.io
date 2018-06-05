---
layout: post
title: Doing PXE installations without control over the DHCP server.
date: '2012-04-29T22:58:00.001+01:00'
author: Niels Basjes
tags:
- Linux
- PXE
---

In many cases there is a desire to do PXE based installations that can be managed by tools like cobbler and puppet. This page describes how this can be accomplished by using a combination of cobbler, dnsmasq and (optionally) puppet.

<h2>Introduction</h2>
<p>I wrote this because I got emails stating that what I described <a href="http://serverfault.com/questions/53116/howto-setup-cobbler-with-pxe-if-you-cant-change-the-dhcp-server">here</a> couldn't be done. They were partially right. It can be done but only with a small patch in cobbler. So I decided to provide <a href="https://github.com/cobbler/cobbler/commit/e338f3502ab550841d9bc36698b329c85ba94047">a fix for cobbler</a> and document the whole thing here.</p>
<p>At the end of this description you'll be able to run a PXE based installation system backed by cobbler without controlling the existing DHCP server. I've had people tell me this can't be done. It can be done and this is how.</p>
<h2>Essential background information on PXE.</h2>
<p>Booting a system using PXE is really using a DHCP broadcast with some additional flags and options.</p>
<p>The booting client needs a few things to continue:</p>
<ol><li>Enough network configuration to connect to the network</li><li>Enough boot information to retrieve and start the code that bootstraps the operating system (usually pxelinux.0).<br /></li></ol>
<p>The trick we will be using is that a single PXE/DHCP request is answered by two "half" answers: one for the network info and one for the boot info.</p>
<p>Now main reason why people say this can't be done is because the most commonly used tools (like the ISC DHCP server) don't support this.</p>
<p>The cool thing is that this is an official part of the PXE standard and with recent versions of dnsmasq we have a tool that actually do this and is very manageable and flexible.</p>
<p>Have a look at <a href="http://www.intel.com/design/archives/wfm/downloads/pxespec.htm">section 1.5.1.1 of the PXE spec</a>; We'll be doing "Separate standard DHCP and redirection service" (=ProxyDHCP).</p>
<h2>The setup</h2>
<h3>Prerequisites</h3>
<ol><li>You need to have a system (may very well be a virtual system) with a common Linux installed. I usually pick the latest CentOS.<br /></li><li>All systems you want to install this way must get a valid IP address from the main DHCP server.</li><li>If the system admins already have a DHCP/PXE server in place that responds to such requests then <strong>STOP NOW</strong>. There is no reliable way to get your PXE server running.</li></ol>
<p>Assuming this is done in an environment where other exist I strongly recommend to play nice.</p>
<p><em>So our setup will ignore any dhcp/pxe requests from unknown systems.</em></p>
<h3>Installing</h3>
<p>You must install</p>
<ul><li>a dnsmasq version 2.51 or newer because we need proxyDHCP support.</li><li>a cobbler version that is newer than 2012-04-29. This is needed because of this <a href="https://github.com/cobbler/cobbler/commit/e338f3502ab550841d9bc36698b329c85ba94047">patch</a><br /></li></ul>
<h3>Configuring<br /></h3>
<p>
Because I'm not providing a 100% installation manual I'll only focus on the basic config settings that need attention.</p>
Simply configure cobbler to control the dns and dhcp for using dnsmasq
<p>.</p>
The config file for dnsmasq is generated using this template
<p>.</p>
<p>I highlighted the important keywords.</p>
<p>/etc/cobbler/dnsmasq.template</p>
<pre><code># Cobbler generated configuration file for dnsmasq
# $date
#

# Usage logging (use this for debug only !)
log-queries
log-async

# resolve.conf .. ?
#no-poll
#enable-dbus
read-ethers
addn-hosts = /var/lib/cobbler/cobbler_hosts

# Be a proxyDHCP server
dhcp-range=10.10.0.0,proxy

# Only respond to clients that are known (i.e present in /etc/ethers)
dhcp-ignore=#known

# Set this (and domain: see below) if you want to have a domain
# automatically added to simple names in a hosts-file.
expand-hosts
domain=example.com,10.10.0.0

# Loads &lt;tftp-root&gt;/pxelinux.0 from dnsmasq TFTP server.
pxe-service=x86PC, "Boot PXELinux (=Cobbler controlled)", pxelinux ,$next_server

# Include the systems that must be "known"
$insert_cobbler_system_definitions</code></pre>
<h2>Running</h2>
<p>Now your should first step is to run this without any systems defined in cobbler. This way you can verify it really ignores unknown requests.</p>
<p>After you verified this you can complete the cobbler installation by importing an operating system and configuring it for installation.</p>
<p>The next step is to define a system that needs to be installed. Now here we do something funny: We only define the MAC address.</p>
<p>We DO NOT define an IP address and there is no use in defining a hostname.</p>
<p>Do <em>cobbler sync</em> and you should be ready to roll.</p>
