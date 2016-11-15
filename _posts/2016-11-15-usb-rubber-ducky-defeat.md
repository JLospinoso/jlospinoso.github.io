---
layout: post
title: Defeating the USB Rubber Ducky with Beamgun
image: /images/beamgun.png
date: 2016-11-15 12:00
tag: Introducing Beamgun, a lightweight USB Rubber Ducky defeat for Windows
categories: [infosec, usb rubber ducky, c#, CLR, WPF, .NET, security]
---
[1]: https://github.com/JLospinoso/beamgun
[2]: https://github.com/JLospinoso/beamgun/issues
[3]: http://usbrubberducky.com/#!index.md
[4]: https://hak5.org
[5]: http://www.usanetwork.com/mrrobot
[6]: https://msdn.microsoft.com/en-us/library/windows/desktop/aa363431(v=vs.85).aspx
[7]: https://msdn.microsoft.com/en-us/library/windows/desktop/ms644990(v=vs.85).aspx
[8]: http://resources.infosecinstitute.com/keyloggers-how-they-work-and-more/
[9]: https://msdn.microsoft.com/en-us/library/mt149843(v=vs.110).aspx
[10]: https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service
[11]: https://msdn.microsoft.com/en-us/library/windows/desktop/aa376875(v=vs.85).aspx

The [USB Rubber Ducky][3] is an awesome little piece of infosec goodness put together by the venerable hackers at
[HAK5][4]. Ever since it was popularized on [Mr. Robot][5], I've wanted to write some software to mitigate the
vulnerability:

[![15 Second Password Hack, Mr. Robot Style](http://img.youtube.com/vi/4kX90HzA0FM/0.jpg)](https://www.youtube.com/watch?v=4kX90HzA0FM "15 Second Password Hack, Mr. Robot Style")

Basically, this little USB device registers itself as a keyboard, waits a bit, then
blasts the victim's computer with a flurry of keystrokes at super-human speed. This requires an
active session (i.e. the user is logged in), although obviously the computer need not be unattended.

The possible effects of this attack are devastating. Generally it is trivial to elevate privileges to
administrator (since you're emulating a user typing), so you can install whatever kind of malware
you'd like, pull larger scripts down from a remote server, or worse.

What, you ask, can men do against such reckless hate?

[![Reckless](http://img.youtube.com/vi/t6qQSll7InQ/0.jpg)](https://www.youtube.com/watch?v=t6qQSll7InQ "Reckless")

Install Beamgun!

![Beamgun Infographic](https://s3.amazonaws.com/net.lospi.beamgun/Readme.png)

You can download and install Beamgun using the simple instructions [on Github][1].


Using Beamgun
===

Beamgun shows up in the system tray whenever you run BeamgunApp.exe. If you install using the MSI installer, this will happen whenever you login. You can click on the icon to view the application's status:

![Beamgun Screenshot](/images/beamgun-armed.PNG)

From here, you can disable Beamgun, e.g. if you are about to insert and remove USB devices frequently, for the next 30 minutes. You can also "Reset" Beamgun if it has alerted, and finally you can force Beamgun to quit by clicking "Exit".

You can also configure the security settings for Beamgun using the two check boxes.
I recommend checking both boxes to get the benefits of both mitigations. Whenever
beamgun detects a USB device, it will only execute the mitigations you have checked.
If you check "Attempt to steal", Beamgun will attemt to steal all of the keystrokes and sink them harmlessly into its window.
If you check "Lock workstation", Beamgun will tell Windows to lock, forcing
you to type your password in to continue.

Beamgun Internals
==

There are a few important Win32 API calls that enable us to do the following:

* monitor for keyboard device insertions so we can alert,
* hook keystrokes so we can see what the attacker tried to do,
* continuously steal keyboard focus to stop the attacker from making progress, and ultimately
* lock the workstation.

Device insertions
===

The [Win32 API][6] gives us the following functions for getting device notifications:

```c
HDEVNOTIFY WINAPI RegisterDeviceNotification(
  _In_ HANDLE hRecipient,
  _In_ LPVOID NotificationFilter,
  _In_ DWORD  Flags
);

BOOL WINAPI UnregisterDeviceNotification(
  _In_ HDEVNOTIFY Handle
);
```

Basically, we're telling Windows that we'd like to get notified about device insertions, and we pass off a handle to our main window. When device-related events happen, Windows alerts us and stuffs a 2-byte code into our callback function to tell us what kind of event it was. See `UsbMonitor.cs` in the `Model` section of `BeamgunApp` for the code that implements all of this.

Hook keystrokes
===

Next, we'd like to hook keystrokes so that when we alert on a USB device insertion, we can record what the attacker was trying to do and present it to the user.

For this, we turn to the tried and true [Windows Hooks APIs][7]:

```c
HHOOK WINAPI SetWindowsHookEx(
  _In_ int       idHook,
  _In_ HOOKPROC  lpfn,
  _In_ HINSTANCE hMod,
  _In_ DWORD     dwThreadId
);

BOOL WINAPI UnhookWindowsHookEx(
  _In_ HHOOK hhk
);
```

Use of these functions is [well documented][8]. We're just using them for good-guy purposes. See `KeyStrokeHooker.cs` for my implementation (again in `Model` of `BeamgunApp`).

Steal focus continuously
===

I chose [Windows Presentation Foundation][9] to build Beamgun, so I used WPF APIs to steal focus:

```cs
public static IInputElement Focus(
	IInputElement element
)
```

The real trick is getting it to run in the thread that's responsible for the window. This can be done with `Dispatcher.Invoke`, as in the following snippet:

```cs
viewModel.StealFocus += () => Dispatcher.Invoke(new MethodInvoker(delegate
{
    Show();
    Activate();
    Topmost = true;
    Keyboard.Focus(this);
}));
```

Here I also call the functions `Show()` and `Activate()` before stealing keyboard focus. This ensures that the Beamgun alert window pops up for the user to see.
See the code-behind in `MainWindow.xaml.cs` of `BeamgunApp` for more details.

Lock the workstation
===
As another (potentially more robust) option, the user can elect to lock the workstation whenever a USB insertion happens. This is safe because we are piggy-backing off the safety mechanisms built into the login screen, e.g. the [local security authority subsystem service][10]. The Win32 API for locking the screen is [as straightforward as it gets][11]:

```cs
BOOL WINAPI LockWorkStation(void);
```

Special considerations
===

Since the Rubber Ducky is a keyboard device, I had to be very careful in implementing Beamgun to try to thwart some countermeasures. I'm sure that some enterprising Rubber Ducky fans will try hard to subvert Beamgun, so I hardened the following features:

* Disabling ALT-F4
* Using custom buttons that do not respond to keyboard events
* Stealing focus on a loop with a very tight interval

Feedback
==
Please [post any issues or bugs][2] you find!
