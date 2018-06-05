---
layout: post
title: Installing Fedora Linux via PXE
date: '2007-12-11T09:24:00.001+01:00'
author: Niels Basjes
tags:
- Linux
- PXE
---

<h2><span id="parent-fieldname-description" class="kssattr-atfieldname-description kssattr-templateId-widgets/textarea kssattr-macro-textarea-field-view inlineEditable">Introduction</span></h2>
<p><span id="parent-fieldname-description" class="kssattr-atfieldname-description kssattr-templateId-widgets/textarea kssattr-macro-textarea-field-view inlineEditable">This page describes the steps I use to install Fedora 8 onto my computers at home without the need for burning any CD's/DVD's.</span></p>
<p>The overall final setup is a 100% network based install of Fedora by using PXE in combination with a HTTP installation.</p>
<p>I've been using this setup for a very long time (since Fedora Core 3) so I know it works .... for me. ;)</p>
<p>This howto was written during the installation of my brand new PC (based on a 64bit Intel Core 2 Quad Q6600 CPU) with Fedora 8. So you'll see x86_64 everywhere. I have done this before with my other computers and it works 99% the same for i686 computers (that 1% is that you must change the name of some of the paths).</p>
<h3>Important remarks</h3>
<p>The system I'm using as my central server at home is a Pentium II at 350Mhz which is happily running Fedora Core 3. Yes, I know this is an ancient version but for my purposes this is good enough. I assume you have a different version so your mileage may vary. One effect of using this old version is that some of the config files I describe will most likely have changed name, place, format over time. <strong>Be prepared to adapt to these differences.</strong></p>
<h3>Overview</h3>
<p>Let's start by giving an overview on what the final installation procedure will look like (this is what we're going to build here).</p>
<ol><li>The client boots from the NIC which supports PXE.</li><li>The NIC sends out a DHCP request with a special 'PXE' flag.</li><li>The DHCP server responds with IP information AND information which image this client must load via TFTP. The image that is specified is <a class="external-link" href="http://syslinux.zytor.com/pxe.php">pxelinux </a>on our TFTP server.<br /></li><li>The client configures the specified IP address.</li><li>The client retrieves the specified image (pxelinux.0) from the TFTP server and starts it.</li><li>The pxelinux software automatically retrieves the configuration with the available list of kernels, images and command line parameters from the TFTP server specific for this system.</li><li>Either automatically or manually a selection is made which image to boot (I have a long list of options here with various Linux distributions and even memtest86).</li><li>The specified Linux kernel (vmlinuz) and initial file system (initrd.img) are downloaded and booted with the specified command line parameters (which ks.cfg must be used !!).</li><li>This base Linux system automatically starts Anaconda which is the Fedora installer.<br /></li><li>One of the command line parameters given to the Linux kernel indicates the location of the ks.cfg configuration file which is used by Anaconda as a configuration what it must do (i.e. what to install from where).</li><li>Anaconda retrieves the next stages of the installation from the specified location (this will be our http server) and starts this software.<br /></li><li>Fedora 8 is now installed either manually or if you wish automatically (as specified in the ks.cfg).<br /></li></ol>
<h3>TIPS</h3>
<ol><li>My most important tip is to try, test and install this by using <a class="external-link" href="http://www.vmware.com/products/server/">VMware Server</a> as the carrier for your test client. It's free, works on both Linux and Windows, and most importantly it supports PXE to boot the system from the network.</li><li>Under the assumption that most networks already have a production
DHCP server that your not allowed to change during development you can
create two virtual machines within your PC and create a completely
isolated host only network with only those two Virtual Machines. In
such a way you can have a fully isolated test environment to try this
out befor you migrate it to a production situation.</li><li>After you have completed the first system by manually selecting the packages you want; the installer writes a ks.cfg file in the home directory of the root user of that system. You can use this new file as the basis for installing multiple ' identical' systems by copying those settings to the ks.cfg used during installation.<br /></li></ol>
<h3>One step at a time</h3>
<p>We're building this setup one step at a time.</p>
<ol><li>Download Fedora 8.</li><li>Setup HTTP for HTTP based installation.</li><li>Test the HTTP based installation by booting from CDROM image.<br /></li><li>Setup PXE/TFTP to bootstrap the installation over the network.</li><li>Test the PXE/HTTP based installation by booting from PXE.<br /></li><li>Install your systems.</li></ol>
<h3>Assumptions</h3>
<p>I'm basing this on a few specific assumptions</p>
<p><strong>Client<br /></strong></p>
<ol><li>
<p>The client is an Intel/AMD kind of computer. I've been doing this on Pentium II/III/IV/D/Core 2 Quad computers and also in VMware "computers". It works on those systems.</p>
</li><li>The network card of the client that is to be installed supports the Preboot eXecution Environment (PXE). PXE is simply support for booting a computer over the network where it receives the information on 'what to boot from where' from the DHCP server. If your NIC doesn't support that then remember that the RIS feature in Windows Server is also based on PXE .... and includes a bootdisk (which is essentially a software based PXE) that allows PXE on systems where the hardware does not support PXE .</li></ol>
<p><strong>Server</strong></p>
<ol><li>You have the ISC DHCP server in your LAN to boot from. Any DHCP that allows special boot options should work.<br /></li><li>On the SAME SYSTEM as the DHCP server you have a TFTP server. This SAME system is because a lot of older PXE clients (including some of the versions of the mentioned RIS diskette) cannot handle
having a different server for the DHCP part and the TFTP part.</li><li>A system with an Apache HTTP server which has enough storage and capacity to serve the entire distribution. <br /></li></ol>
<h2>Download Fedora 8<br /><span id="parent-fieldname-description" class="kssattr-atfieldname-description kssattr-templateId-widgets/textarea kssattr-macro-textarea-field-view inlineEditable"></span></h2>
<h3>Which one ?</h3>
<p>Now you can do one of two things:</p>
<ul><li>You download <strong>'Everything'</strong> which contains <strong>ALL available RPMs</strong> for Fedora 8 entails about 12GB of data.</li><li>You download <strong>'Fedora'</strong> which is contains the <strong>normal set of RPMs</strong> for Fedora 8 entails about 4GB of data.</li></ul>
<p>You choose, I always use 'the big one' because it makes live easier in the long run.</p>
<p>&nbsp;</p>
<h3><span id="parent-fieldname-description" class="kssattr-atfieldname-description kssattr-templateId-widgets/textarea kssattr-macro-textarea-field-view inlineEditable">Mirror Fedora 8 'Everything' to your local server<br /></span></h3>
<p>The first this you need to do is mirror the right part of the Fedora 8 distribution.</p>
<p>First of all you go to <a class="external-link" href="http://mirrors.fedoraproject.org/">http://mirrors.fedoraproject.org/</a> and select the mirror site that is best for your situation.</p>
<p>&nbsp;</p>
<p>Now the right part is the directory which is under&nbsp;&nbsp; <strong>releases/8/Everything/x86_64/os/</strong></p>
I rsync it all to:
<pre>/opt/deploy/install/fedora/releases/8/Everything/x86_64/os</pre>
<p>The strange thing is that 'Everything' does not have the PXE images .... So I also rsync releases/8/Fedora/x86_64/os/images</p>
<pre>/opt/deploy/install/fedora/releases/8/Fedora/x86_64/os/images</pre>
<p>Now before we continue we must copy the entire images directory from 'Fedora' to 'Everything' ( I've reported this issue in the Fedora Bugzilla as <a class="external-link" href="https://bugzilla.redhat.com/show_bug.cgi?id=419351">issue 419351</a> )<br /><span class="visualHighlight"></span></p>
<p>So I copy the</p>
<pre>/opt/deploy/install/fedora/releases/8/Fedora/x86_64/os/images</pre>
<p><span class="visualHighlight"></span></p>
<p>to the new place<br /><span class="visualHighlight"></span></p>
<pre>/opt/deploy/install/fedora/releases/8/Everything/x86_64/os/images</pre>
<p><span class="visualHighlight"><br /></span></p>
<h2>HTTP installation</h2>
<h3>DNS <br /></h3>
<p>I chose to create an additional hostname in my DNS with the name
'install'. So I can simply specify http://install/.... as the download
path in the Fedora installer anaconda.<br />It depends on your DNS server how you configure this.</p>
<h3>Setup HTTP server for HTTP based installation</h3>
<p>The first thing we need to do is make the stuff we just downloaded available via HTTP.<br />This is simply done by configuring the webserver in such a way that these files have a URL under which they can be found.</p>
<p>&nbsp;</p>
<p>My /etc/httpd/conf.d/install.conf looks like this:</p>
<pre>&lt;Directory /opt/deploy/install &gt;
&nbsp;&nbsp;&nbsp; AllowOverride All
 &nbsp;&nbsp; AcceptPathInfo On
&nbsp;&nbsp;&nbsp; #Options Indexes FollowSymLinks
&nbsp;&nbsp;&nbsp; Options Indexes SymLinksIfOwnerMatch FollowSymLinks
&lt;/Directory&gt;

&lt;VirtualHost *:80&gt;
&nbsp;&nbsp;&nbsp; DocumentRoot /opt/deploy/install/
&nbsp;&nbsp;&nbsp; ServerName install
&nbsp;&nbsp;&nbsp; UseCanonicalName on
&lt;/VirtualHost&gt;</pre>
<h3>Testing the HTTP setup&nbsp;</h3>
<p>First try to open this URL in your browser:</p>
<p><a class="external-link" href="http://install/fedora/releases/8/Everything/x86_64/os/RELEASE-NOTES-en_US.html">http://install/fedora/releases/8/Everything/x86_64/os/RELEASE-NOTES-en_US.html</a></p>
<p>This should provide you with the release notes on your screen.</p>
<p>Now startup a VMware with the boot.iso ( located in /opt/deploy/install/fedora/releases/8/Fedora/x86_64/os/images ) as the CDROM image.</p>
<p>Choose installation method HTTP</p>
<p>Website: install</p>
<p>Fedora directory: fedora/releases/8/Fedora/x86_64/os</p>
<p>Now when you continue the graphical installer should load.</p>
<p>&nbsp;</p>
<p>If it does then this part works.</p>
<p>&nbsp;</p>
<h2>PXE installation bootstrap</h2>
<p>&nbsp;</p>
<h3>Configure the TFTP server</h3>
<p>
I'm using tftp-server-0.39-1</p>
<p>
My config file is called /etc/xinetd.d/tftp and contains this:</p>
<pre>service tftp
{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; socket_type&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = dgram
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; protocol&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = udp
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; wait&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = yes
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; user&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = root
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; server&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = /usr/sbin/in.tftpd
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; server_args&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = -s /opt/deploy/install -v -v
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; disable&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = no
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; per_source&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = 11
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cps&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = 100 2
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; flags&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; = IPv4
}</pre>
<p>&nbsp;</p>
<h3>Setup PXELinux</h3>
<p>I downloaded the latest version from http://www.kernel.org/pub/linux/utils/boot/syslinux/syslinux-3.53.tar.gz and installed the pxelinux.0 in there under my Fedora tree.</p>
<p>cp pxelinux.0 /opt/deploy/install/fedora/releases/8/Everything/x86_64/</p>
<p><strong><em>IMPORTANT</em><br />If traversing the path /opt/deploy/install/fedora/releases/8/Everything/x86_64/ to find pxelinux.0 contains a symbolic link then the TFTP server will simply not be able to send the provided file.</strong> This fact may be specific to the TFTP server version I'm using but then again , it may be the same in current versions.</p>
<p>Now my config file</p>
<pre>/opt/deploy/install/fedora/releases/8/Everything/x86_64/pxelinux.cfg/default</pre>
<p>looks like this:</p>
<pre>DEFAULT fedora8
TIMEOUT 0

PROMPT 0

LABEL fedora8
KERNEL os/images/pxeboot/vmlinuz kssendmac
APPEND initrd=os/images/pxeboot/initrd.img ks=http://install/fedora/releases/8/Everything/x86_64/ks.cf</pre>
<p>As you can see we refer to the ks.cfg under the specified URL.</p>
<p>The paths to the kernel and images are relative to the directory where pxelinux.0 is located.</p>
<h3>Create a ks.cfg to install from HTTP</h3>
<p>The ks.cfg file is the config file that anaconda uses to automate
specific selections in the install process. You can fully automate the
installation using the ks.cfg</p>
<p>I prefer for my situation to simply let the system figure out where
the installation files are located (on my http://install/ server).</p>
<p>So this file /opt/deploy/install/fedora/releases/8/Everything/x86_64/ks.cfg&nbsp; simply contains:</p>
<pre>url --url http://install/fedora/releases/8/Everything/x86_64/os/</pre>
<p>Check using your normal browser if you can download <a class="external-link" href="http://install/fedora/releases/8/Everything/x86_64/ks.cfg">http://install/fedora/releases/8/Everything/x86_64/ks.cfg</a></p>
<h3><span id="parent-fieldname-description" class="kssattr-atfieldname-description kssattr-templateId-widgets/textarea kssattr-macro-textarea-field-view inlineEditable">Setup the DHCP server so it supports PXE for your host.<br /></span></h3>
Now my personal setup uses the DHCP LDAP configuration feature created by <a class="external-link" href="http://home.ntelos.net/~masneyb/">Brian Masney</a>
<p>This is much too much overkill for most situations so I'm describing the 'normal setup'.</p>
Check the <a class="external-link" href="http://syslinux.zytor.com/pxe.php">Official pxelinux page</a> because they have a lot of examples on this topic.
<p><br />The relevant bit is that you have the following parameters for the systems your going to install:</p>
<pre>next-server "install"
filename "/fedora/releases/8/Everything/x86_64/pxelinux.0"</pre>
<p>Note that this path is relative to the root specified for the tftp server</p>
<p>&nbsp;</p>
<h2>Testing the PXE/HTTP setup&nbsp;</h2>
<p>Now start your VMware machine again and press F12 during the BIOS phase.</p>
<p>&nbsp;<img class="image-inline" src="/assets/images/2007-12-07-VMWare BIOS.png" alt="VMware BIOS is loading" /></p>
<p>The Virtual Machine will boot from the network and should load the information from the server.</p>
<p><img class="image-inline" src="/assets/images/2007-12-07-Booting using PXE.png" alt="Booting using PXE" /></p>
<p>And show stage 1 of the installation procedure.</p>
<p><img class="image-inline" src="/assets/images/2007-12-07-Fedora 8 Stage 1.png" alt="Fedora 8 installer stage 1" />&nbsp;</p>
<p>After selecting language and keyboard it should load images/stage2.img</p>
<p>and show the graphical installer.</p>
<p><img class="image-inline" src="/assets/images/2007-12-07-Fedora 8 Stage 2.png" alt="Fedora 8 installer stage 2" /></p>
<p>&nbsp;</p>
<div align="center"><strong><span class="visualHighlight">SUCCESS we have now started the Fedora 8 installer via PXE.</span></strong></div>
<p>&nbsp;</p>
<p>&nbsp;</p>
<a href="http://creativecommons.org/licenses/by-nc-sa/3.0/nl/" rel="license">
<img src="http://i.creativecommons.org/l/by-nc-sa/3.0/nl/88x31.png" alt="Creative Commons License" />
</a>

