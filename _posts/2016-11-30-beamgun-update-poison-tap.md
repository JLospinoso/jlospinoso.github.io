---
layout: post
title: Defeating PoisonTaps and other Rogue Network Adapters with Beamgun
image: /images/lanturtle.png
date: 2016-11-30 20:00
tag: New and improved Beamgun now defends against USB network adapters and mass storage!
categories: [infosec, usb rubber ducky, lan turtle, c#, CLR, WPF, .NET, security]
---
[1]: https://github.com/JLospinoso/beamgun
[2]: https://samy.pl/poisontap/
[3]: http://usbrubberducky.com/#!index.md
[4]: https://hak5.org
[5]: https://lanturtle.com/
[6]: https://www.hak5.org/episodes/hak5-1824
[7]: https://www.hak5.org/episodes/threatwire/stealing-cookies-from-sleeping-pcs-icloud-call-history-android-updates-unencrypted-threat-wire
[8]: https://twit.tv/shows/twit-bits/episodes/3359?autostart=false
[9]: https://room362.com/post/2016/snagging-creds-from-locked-machines/
[10]: https://inversepath.com/usbarmory
[11]: https://www.raspberrypi.org/products/pi-zero/
[12]: https://jlospinoso.github.io/infosec/usb%20rubber%20ducky/c%23/clr/wpf/.net/security/2016/11/15/usb-rubber-ducky-defeat.html
[13]: https://gallery.technet.microsoft.com/Enabling-USB-Mass-Storage-c0b19e62
[14]: https://msdn.microsoft.com/en-us/library/aa394582(v=vs.85).aspx
[15]: https://msdn.microsoft.com/en-us/library/aa392902(v=vs.85).aspx
[16]: https://www.hak5.org/episodes/season-21/hak5-2112-stealing-files-with-the-usb-rubber-ducky
[17]: https://blogs.technet.microsoft.com/danstolts/2009/01/disable-adding-usb-drive-and-memory-sticks-via-group-policy-and-group-policy-preferences/
[18]: https://github.com/JLospinoso/beamgun/issues

Over the past few months, security researchers like [Rob Fuller][9] and
[Samy Kamkar][2] have been exploring vulnerabilities associated with the
implicit trust Windows gives to USB network adapters. Wielding devices like
the [USB Armory][10], the [LAN Turtle][5], and even the $5 [Raspberry Pi Zero][11],
Rob and Samy have demonstrated how simply inserting one of these rogue USB devices
into a Windows machine--whether it's locked or not--can affect [serious compromise
of logged in users][6], including:

* storing HTTP cookies and sessions,
* persistence to an attacker as an internal router,
* gives persistent HTTP cache poisoning,
* forcing the user to make HTTP requests and proxy back responses

Aside from logging out of a computer each time you step away or epoxy-gluing your
USB ports shut, [there aren't very many good options for mitigating against this
kind of attack][8].

Beamgun
==
Last month I released [Beamgun][12], an [open-source][1] Windows application that
mitigates against another kind
of rogue USB device, the [USB Rubber Ducky][3]. In principle, both of these devices
rely on the implicit trust Windows gives to physical machine access. With the Ducky,
Windows gives full trust to a newly-inserted USB keyboard. The Ducky then blasts the
target computer with a rapid volley of pre-programmed keystrokes (you can imagine
  how bad that is). Windows administrators are not able to disable USB devices without physically
disabling ports (note that you _are_ able to [disable USB mass storage][13]--more on that
later).

I replaced the Win32-API-based `RegisterDeviceNotification` approach with [Windows Management
Instrumentation][14] approach. We first construct a `WqlEventQuery`:

```cs
var keyboardQuery = new WqlEventQuery("__InstanceCreationEvent", new TimeSpan(0, 0, 1),
  "TargetInstance isa \"Win32_Keyboard\"");
```

and then we pass this query to a `ManagementEventWatcher`:

```cs
_watcher = new ManagementEventWatcher(keyboardQuery);
```

This new `_watcher` has an `EventArrived` event handler where we can register arbitrary callbacks. A major improvement to the WMI approach is that we only get callbacks for keyboards rather than all USB devices!
The upshot of this is that there are far fewer false-positives. Before, Beamgun would alert on any USB device,
now it's just keyboards.

Ok, ok, but what about PoisonTap?!
==

Registering for new network adapters (as in ALL network adapters, not just USB!) is as simple as changing the
[WQL][15] to our new target:

```cs
var networkQuery = new WqlEventQuery("__InstanceCreationEvent", new TimeSpan(0, 0, 1),
  "TargetInstance isa \"Win32_NetworkAdapter\"");
```

The new challenger here is that just locking the workstation is no longer a sufficient mitigation. With rogue
keyboard devices, locking the workstation blocks the attack (unless the attacker knows your password). PoisonTap
is a particularly nasty attack because it can work even when your computer is locked.

Fortunately, we can disable the new network adapter immediately upon receiving notification via WMI:

```cs
_watcher.EventArrived += (caller, args) => {
  var obj = (ManagementBaseObject)args.NewEvent["TargetInstance"];
  var query = $"SELECT * FROM Win32_NetworkAdapter WHERE DeviceID = \"{obj["DeviceID"]}\"";
  var searcher = new ManagementObjectSearcher(query);
  foreach (var item in searcher.Get())
  {
      var managementObject = (ManagementObject)item;
      try
      {
          var disableCode = (uint)managementObject.InvokeMethod("Disable", null);
          // ...
          return;
      }
      catch (ManagementException e)
      {
          // ...
      }
  }
};
```

So again, while we cannot tell Windows to block these new devices explicitly, we _can_ listen for the new devices
with callbacks and take action when they are detected.

USB Mass Storage Devices
==
Things can get particularly crappy for the target when an attacker adds storage into the mix.
[Darren Kitchen][16] recently demonstrated some methods for slurping files directly off a target
with the USB Rubber Ducky. This kind of attack completely bypasses any kind of firewall/perimeter monitoring that may be in place. All of the document stealing happens right on the spot.

Supposing that Beamgun's workstation lock was evaded somehow, there's an
added layer of protection we can add basically for free. The one USB policy that Windows _does_ support
is to [disable USB Mass Storage devices][13] with the following registry key:

```
HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\usbstor\\Start
```

If it's set to three, USB is enabled--four, and it's disabled. You can even [set it with group
policy][17]. Since it's just tweaking a registry entry, I've added it to Beamgun as an option.
Hey--it's the one rogue-USB-device-mitigation that Windows actually ships with, so why not use it?

Updates
==
Beamgun can now check to see if new versions have been released by periodically polling

```
https://s3.amazonaws.com/net.lospi.beamgun/version.json
```

If it finds it's out of date, it will write a friendly message into the log. If you don't want this, just set this registry entry to "False":

```
HKEY_CURRENT_USER\\SOFTWARE\\Beamgun\\CheckForUpdates
```

Installing Beamgun
==
You can get links to install beamgun, clone the repo, etc. at [https://github.com/JLospinoso/beamgun][1].

Feedback
==
Please [post any issues or bugs][2] you find!
