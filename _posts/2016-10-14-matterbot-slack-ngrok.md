---
layout: post
title: Setting Up a Matterbot with Slack via ngrok
image: /images/slackbot.jpg
date: 2016-10-14 12:00
tag: Running a Matterbot on Slack with ngrok
categories: [c++, web, rest, mattermost, software, developing, ngrok]
---
[1]: https://github.com/JLospinoso/matterbot
[2]: https://api.slack.com/
[3]: https://ngrok.com/
[4]: https://jlospinoso.github.io/c++/web/rest/mattermost/software/developing/2016/06/14/matterbot.html
[5]: https://en.wikipedia.org/wiki/Network_address_translation
[6]: https://slack.com/
[7]: https://github.com/JLospinoso/matterbot/issues
[8]: https://api.slack.com/incoming-webhooks

When [matterbot][1] was [introduced][4], we configured it to work with an instance of
Mattermost that we hosted. In this post, we'll get it working with [Slack][6], which
will already hosted be for us. The major challenge is that your bot may not be routable
from Slack's servers (i.e. the internet). If you are behind a firewall, [NAT][5]ed behind
a router, etc., you will be able to `POST` messages into chatrooms via [incoming webhooks][8]
but you won't receive messages from Slack when people try to trigger your bot. [ngrok][3] is
a beautiful solution for situations like this. After signing up and downloading a client, you'll
get a subdomain that will tunnel traffic to you (even if you aren't routable!)

# ngrok
Setting up an account with [ngrok][3] is simple and free. Just follow the instructions, then download the client. I put the client binary on my PATH so that I can invoke it from anywhere in my command prompt (I suggest that you do the same).

Once you've signed up for an account and you've downloaded the client, you'll need to install your authtoken (this will be available from your dashboard.) Just run ngrok with the issued token:

```bash
ngrok authtoken THISISMYSECRETTOKEN123ABC
```

Now, you'll be able to expose any local port you'd like to the internet (be careful!) We are about to bind Matterbot to local port 8000, so all you need to do is issue the command

```bash
ngrok http 8000
```

You'll end up with some output that looks like this:

```
ngrok by @inconshreveable                  (Ctrl+C to quit)

Session Status                online
Account                       Josh Lospinoso (Plan: Free)
Version                       2.1.14
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://88319c22.ngrok.io -> localhost:8000
Forwarding                    https://88319c22.ngrok.io -> localhost:8000

Connections                   ttl     opn     rt1     rt5     p50     p90
                              0       0       0.00    0.00    0.00    0.00
```

You can navigate to http://127.0.0.1:4040 in your browser, and you'll be able
to interact with ngrok through a nice web app. Notice that you've now got an internet-routable domain that is mapped to a local
port on your machine! In my case, it's http://88319c22.ngrok.io.

Unless you have a paid plan with ngrok, this subdomain will change whenever you
restart ngrok. Keep this in mind as you're developing!

# Slack

Now, sign up for a [Slack][6] account. You'll need administrative privileges to enable webhooks for Matterbot, so create a new team (or continue along with a team that you have administrative privileges for).

1. From the team homepage, click the top-right icon to get into the drop-down menu.
2. Select "Apps & Integrations". Alternatively, just point your browser to https://MY-TEAM-NAME.slack.com/apps where
MY-TEAM-NAME is the appropriate subdomain.
3. In the search bar, type "Incoming WebHooks" and click on the Incoming WebHooks icon when it pops up.
4. Click on "Add Configuration"
5. Choose the channel you'd like the Matterbot to interact with in the drop-down menu.
6. Click on "Add Incoming WebHooks integration."
7. Surf right over all the setup instructions (Matterbot will handle all of these details for you!) *Save the "Webhook URL". You'll need this when setting up your Matterbot.*
8. Optionally, you can give your bot a descriptive label, customize its name, give it a custom icon, etc.
9. When you're done customizing your bot, click on "Save Settings."

We are going to follow a similar pattern for the Outgoing WebHooks; go back to the "Apps & Integrations" search, and continue:
10. In the search bar, type "Outgoing WebHooks" and click on the Outgoing WebHooks icon when it pops up.
11. Click on "Add Outgoing WebHooks Configuration"
12. Choose the channel you'd like the Matterbot to interact with in the drop-down menu. This should match the channel you selected for Incoming WebHooks.
13. As in the Mattermost case, you'll need to choose some trigger words that your bot will respond to. Put these in the "Trigger Word(s)" textbox and separate them with a comma.
14. In the "URL(s)" box, put your internet-routable ngrok URL. In my case, it is https://88319c22.ngrok.io. Note again that if you restart ngrok on your machine, you will need to change the settings of your Outgoing WebHooks!
15. *Save the "Token". You'll need this when setting up your Matterbot.*
8. Optionally, you can give your bot a descriptive label, customize its name, give it a custom icon, etc.
9. When you're done customizing your bot, click on "Save Settings."

# Matterbot

You are now ready to build Matterbot and wire it into Slack. Pull down the latest version from [github][1]:

```bash
git clone git@github.com:JLospinoso/matterbot.git
```

Open `matterbot/Matterbot.sln` in Visual Studio. In the MatterbotSample project,
open `main.cpp` in Source Files. You'll notice several `TODO` sections to fill in.
The only requirement to get things up and running is to paste your routes and tokens:

```cpp
//TODO: Put your own routes and tokens here
wstring mattermost_url = L"https://hooks.slack.com", // URL to the incoming webhook for Mattermost/Slack
  incoming_hook_route = L"services/AAAAAAAAA/BBBBBBBBB/CCCCCCCCCCCCCCCCCCCCCCCC", // Route
  outgoing_hook_route = L"http://127.0.0.1:8000/", // URL of the box running matterbot
  outgoing_hook_token = L"CCCCCCCCCCCCCCCCCCCCCCCC";
```

Take the route portion of the URL you copied when setting up your Incoming WebHooks and put this into
the `incoming_hook_route` variable. Put your Outgoing WebHook token into `outgoing_hook_token`. If you set up ngrok to listen on `:8000` then you are all set. If not, modify `outgoing_hook_route`
to bind to the correct port.

# Testing
Navigate to the [slack][6] channel that you've configured Matterbot to interact with. Using your trigger word
for the bot (mine is `matterbot`), broadcast this message into the channel:

```bash
matterbot help
```

If all is well, you should receive a helpful message from your bot telling you that it supports the
`echo` command. You can follow up by issuing an echo:

```bash
matterbot echo Hello, world!
```

Your matterbot should echo the given message back to you.

See the [original post][5] on Matterbot to see how to write your own functionality into the bot!

Feedback
==
Please [post any issues or bugs][7] you find!
