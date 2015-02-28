---
layout: post
title: Configuring Wireshark to collect without elevated privileges on Ubuntu 14
date: 2015-02-11 03:49
author: jalospinoso
comments: true
categories: [software]
---
Wireshark can be installed in Ubuntu very easily:

<code>sudo apt-get install libcap2-bin wireshark</code>

Unfortunately, it is usually not possible to capture packets on any interfaces without running wireshark with elevated privileges (e.g. as root):

<code>sudo wireshark &amp;</code>

There is a series of steps to follow that implement "Privilege Separation," effectively allowing the Wireshark GUI to run as a normal user while dumpcap (which is collecting the packets from your interfaces) runs with the elevated privileges required to sniff. First, try:

<code>sudo dpkg-reconfigure wireshark-common</code>

And select "Yes" when prompted as to whether non-superusers should be able to capture packets.

In the event that this doesn't do the trick, issue the following series of commands, which will configure the dumpcap executable to run. Be sure to include YOUR_USER_NAME where indicated below:

<code>sudo chgrp YOUR_USER_NAME /usr/bin/dumpcap
sudo chmod 750 /usr/bin/dumpcap
sudo setcap cap_net_raw,cap_net_admin+eip /usr/bin/dumpcap</code>

See the following for more assistance should you still have issues:
<ul>
	<li><a href="http://wiki.wireshark.org/CaptureSetup/CapturePrivileges">http://wiki.wireshark.org/CaptureSetup/CapturePrivileges</a></li>
	<li><a href="http://ubuntuforums.org/showthread.php?t=2039978">http://ubuntuforums.org/showthread.php?t=2039978</a></li>
	<li><a href="https://ask.wireshark.org/questions/7976/wireshark-setup-linux-for-nonroot-user">https://ask.wireshark.org/questions/7976/wireshark-setup-linux-for-nonroot-user</a></li>
	<li><a href="http://packetlife.net/blog/2010/mar/19/sniffing-wireshark-non-root-user/">http://packetlife.net/blog/2010/mar/19/sniffing-wireshark-non-root-user/</a></li>
</ul>
