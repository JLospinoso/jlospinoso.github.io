---
layout: post
title: Configuring Wireshark on Ubuntu 14
date: 2015-02-11 03:49
image: /images/wireshark.png
tag: Wireshark can be configured to work without having root privileges
categories: [software, wireshark, networks, ubuntu]
---

Wireshark can be installed in Ubuntu very easily:

	sudo apt-get install libcap2-bin wireshark

Unfortunately, it is usually not possible to capture packets on any interfaces without running wireshark with elevated privileges (e.g. as root):

	sudo wireshark &

There is a series of steps to follow that implement *Privilege Separation*, effectively allowing the Wireshark GUI to run as a normal user while dumpcap (which is collecting the packets from your interfaces) runs with the elevated privileges required to sniff. First, try:

	sudo dpkg-reconfigure wireshark-common

And select *Yes* when prompted as to whether non-superusers should be able to capture packets.

In the event that this doesn't do the trick, issue the following series of commands, which will configure the dumpcap executable to run. Be sure to include `YOUR_USER_NAME` where indicated below:

	sudo chgrp YOUR_USER_NAME /usr/bin/dumpcap
	sudo chmod 750 /usr/bin/dumpcap
	sudo setcap cap_net_raw,cap_net_admin+eip /usr/bin/dumpcap

See the following for more assistance should you still have issues:

* <http://wiki.wireshark.org/CaptureSetup/CapturePrivileges>
* <http://ubuntuforums.org/showthread.php?t=2039978>
* <https://ask.wireshark.org/questions/7976/wireshark-setup-linux-for-nonroot-user>
* <http://packetlife.net/blog/2010/mar/19/sniffing-wireshark-non-root-user/>

