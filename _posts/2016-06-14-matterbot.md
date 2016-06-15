---
layout: post
title: Matterbot
image: /images/chatbot.jpg
date: 2016-06-14 21:00
tag: A C++ Framework for Creating Simple Mattermost/Slack Bots
categories: [c++, web, rest, mattermost, software, developing]
---
[1]: https://github.com/JLospinoso/matterbot
[2]: http://www.mattermost.org/webhooks/
[3]: https://api.slack.com/
[4]: http://webhooks.us/
[5]: https://github.com/Microsoft/cpprestsdk
[6]: https://jlospinoso.github.io/subliminal-channel/twitter/poco/cryptography/c++/developing/software/2016/02/06/twitter-subliminal.html
[7]: http://pocoproject.org/
[8]: https://docker.com/
[9]: https://hub.docker.com/r/mattermost/mattermost-preview/
[10]: http://docs.mattermost.com/help/getting-started/creating-teams.html
[11]: http://docs.mattermost.com/developer/webhooks-incoming.html
[12]: http://www.c2.com/cgi/wiki?PimplIdiom
[13]: https://jlospinoso.github.io/c++/developing/software/visual%20studio/2015/03/11/lambdas-and-cpp11.html
[14]: https://github.com/JLospinoso/matterbot/issues
[15]: https://en.wikipedia.org/wiki/Media_type

[matterbot][1] is a framework for making Mattermost/Slack bots.
It uses the [Webhooks][4] APIs exposed by both [Mattermost][2] and [Slack][3] to
send and receive messages. In the [twitter-subliminal][6] project, we tried out [Poco Libraries][7] to handle our
web communications. matterbot tries out something different: Microsoft's [C++ REST SDK][5] a.k.a. Casablanca.

Getting started
==
First, you'll need a running Mattermost/Slack service. If you already have one, skip ahead to the next section. For the remainder of the post,
we use Mattermost, but the webhook APIs are compatible.

I highly recommend using [docker][8] while you are developing your bot. You can set up the service with this one-liner (*Note: You should NOT use this command to set up a production instance!*):

```sh
$ docker run --name mattermost-preview -d --publish 8065:8065 mattermost/mattermost-preview
f25bbb6897ca0a1e9a0313cef5c31a6864647ed18e593a6736925af83f8b2523
```

You can see your docker container's status with `docker ps`:

```
$ docker ps
CONTAINER ID        IMAGE                           COMMAND                  CREATED
f25bbb6897ca        mattermost/mattermost-preview   "/bin/sh -c ./docker-"   3 minutes ago
     STATUS              PORTS                              NAMES
     Up 3 minutes        3306/tcp, 0.0.0.0:8065->8065/tcp   mattermost-preview
```

You'll note that the container has mapped the docker host's port 8065. If you point a web
browser at *http://[my-docker-ip]:8065*, you should be greeted with a friendly user
signup screen. Follow the on-screen instructions or [see the docs][10] to set up a
username and a team.

Setting up Webhooks
==
Once you've set up a team, you'll need to enable webhooks. Full documentation is [here][11], but the gist is that you'll need to go to the System Console and click
"Enable Incoming Webhooks" and "Enable Outgoing Webhooks". The System Console can
be accessed by clicking on the three vertical dots in the upper-left of the team site.
It should be the seventh entry from the top in the drop-down menu that appears.

Go back to your team site. Webhooks are created at the team-level. Click again on the
three vertical dots, and this time an "Integrations" option should appear. Click it.

You'll need to create the webhooks individually. First, setup the Incoming Webhook.
Click on the Incoming Webhook icon, then click "Add Incoming Webhook". The display name
and description do not correspond to what will actually appear in the chat window--they
are just for administrative accounting purposes. Select the channel that you want the bot
to post messages into, then click save. Take note of the resulting URL, e.g.:

```
http://192.168.1.2:8065/hooks/ktjckuh9ptrnmgoiunadsitgmc
```

Next, setup an Outgoing Webhook by clicking on the "Outgoing Webhooks" option under the
"Integrations" header at the top of the screen. Click on "Add Outgoing Webhook". The
display name and description are again just for administrative accounting purposes. The
"Channel" is actually optional--if you want the bot to listen to all channels, don't
select anything here. The trigger words are important--Mattermost won't send your bot a message unless it begins
with one of the trigger words listed here.

Finally, you'll need to specify your callback URL. Locate the IP of the machine you'll
be running your matterbot from, and enter it in the "Callback URLs" box, e.g.:

```
http://192.168.1.3
```

Also take note of the token that gets created for your outgoing webhook, e.g.:

```
Token: omy7rqidk3dqdqky39yssm4bao
```

Configure your firewall!
==
Your bot is going to need to listen to port 80. Configure your firewall to allow this.
If you are just running Mattermost in a docker container, you *may* be able to get away
with default firewall rules.

Building an example bot
==
Pull down matterbot from [github][1]:

```sh
git clone git@github.com:JLospinoso/matterbot.git
```

Open up Visual Studio _as an Administrator_ (so that you can bind to a local port
when debugging). There are two projects in the solution:

* Matterbot is the project containing the matterbot (static) library.
* MatterbotSample is the project containing a sample bot

Both libraries require that NuGet has successfully installed the C++ REST SDK.
Right click on "References" > "Manage NuGet Packages" > "Installed" and make
sure that version 2.8.0 is correctly installed.

MatterbotSample contains one file, `main.cpp`, but it illustrates the main features
of the library. In `main()`, we create a bot by injecting four parameters:

```cpp
wstring mattermost_url = L"http://192.168.4.177:8065/",
	incoming_hook_token = L"ktjckuh9ptrnmgoiunadsitgmc",
	outgoing_hook_route = L"http://192.168.4.99/",
	outgoing_hook_token = L"omy7rqidk3dqdqky39yssm4bao";
//...
Matterbot bot(mattermost_url, incoming_hook_token, outgoing_hook_route, outgoing_hook_token);
```

These parameters are self explanatory--we set up all the webhooks in the previous section,
and you are providing all of the route and token information to the framework here.

Once the bot has been initialized, you can post messages to the channel specified in the
Incoming Webhooks by using the following:

```cpp
bot.post_message(L"Bot is up.");
```

Easy peasy. But the interesting stuff is in implementing commands. Commands are
routines that the bot will run when prompted. These routines give a response, which
is posted into the same channel as `post_message`. Implementing commands is super
simple. Just inherit `ICommand`. MatterbotSample gives an example `echo` command:

```cpp
class EchoCommand : public ICommand {
public:
	wstring get_name() override {
		return L"echo";
	}

	wstring get_help() override {
		return L"`echo [MESSAGE]`\n===\n`echo` will respond with whatever
		message you give it.";
	}

	wstring handle_command(wstring team, wstring channel, wstring user,
		wstring command_text) override {
		return command_text;
	}
};
```

There are three functions in `ICommand`:

* `get_name` is the command name that the bot will look for when it receives an outgoing webhook. So if we registered our bot to listen for `#chatbot`, then `EchoCommand` would get a callback when someone typed `#chatbot echo Hello, world!`.
* `get_help` is an optionally-markdown flavored response that explains to the user how the command works. More on help in a moment.
* `handle_command` is a callback whenever Mattermost/Slack alerts us via outgoing webhook that someone has triggered the bot. We get information like the team, channel, and user, as well as the full command text. The `wstring` result returned by `handle_command` is sent back by the bot.

You'll register all of your commands with the bot by passing it a shared pointer:

```cpp
bot.register_command(make_shared<EchoCommand>());
```

When a user prompts your bot for help, they will get a listing of all commands supported by the bot, e.g.

```
user > #chatbot help

bot > Supported commands
bot > echo [MESSAGE]
bot > echo will respond with whatever message you give it.
bot > checkbuild [build_id]
bot > checkbuild will retrieve the status of the build with build_id
bot > haiku
bot > bot will send you the haiku of the day
...
```

You can also ask for help about a specific command, e.g.

```
user > #chatbot help echo

bot > echo [MESSAGE]
bot > echo will respond with whatever message you give it.
```

The default logger will push messages from matterbot into `wclog`, but you can
customize this behavior by implementing your own `ILogger`:

```cpp
class CustomLogger : public ILogger {
	void info(const wstring &msg) override {
		wcout << "INFO: " << msg;
	}
	void warn(const wstring &msg) override {
		wcout << "WARN: " << msg;
	}
	void error(const wstring &msg) override {
		wcerr << "INFO: " << msg;
	}
};
```

Overwrite the default logger with `set_logger`:

```cpp
bot.set_logger(make_unique<CustomLogger>());
```

One other feature of Matterbot is that it accepts `GET` requests at the same URL
as the Outgoing Webhook URL (this comes basically for free since we need to) bind to the port anyway. The default response is a status webpage that gives basic statistics about the bot:

```
MattermostBot Status

Web requests served: 17
Messages posted: 165
Commands served: 2135
Supported commands:
*checkbuild
*echo
*haiku
...
```

Implementation details
==
That's really all you need to get started building your own bot, but in case you would like to repurpose some (or all) of the matterbot source, here's a brief overview of how the pieces fit together.

The `Matterbot.h` header is designed around the [PIMPL idiom][12]. The practical upshot of this design choice is that `Matterbot.h` is the only non-standard library header that gets imported:

```cpp
#pragma once
#include <memory>
#include <string>

namespace lospi {
	class ILogger {
//...
	};

	class ICommand {
	public:
//...
	};

	class MatterbotImpl;
	class Matterbot {
	public:
//...
	private:
		std::shared_ptr<MatterbotImpl> impl;
	};
}
```

This is helpful because (a) changing implementation details does not necessarily require a recompiling of classes depending on `Matterbot`, and (b) compile times are generally faster due to less includes.

The low level details of translating HTTP semantics into C++ classes are all handled by the `MattermostWebhooks` class. If you wanted to write a much more involved bot (or bot framework), you could begin with this class to build on top:

```cpp
class MattermostWebhooks
{
public:
	MattermostWebhooks(const std::wstring &mattermost_url,
		const std::wstring &incoming_hook_token,
		const std::wstring &outgoing_hook_route,
		const std::wstring &outgoing_hook_token);
	~MattermostWebhooks();
	void post_message(const std::wstring &message);
	void register_message_handler(const std::function<std::wstring(const Message&)>
		&message_handler);
	void register_web_handler(const std::function<WebResponse()> &web_handler);
	void listen();
	void die();
private:
//...
};
```

For outgoing traffic from Mattermost/Slack, you can register [a std::function][13] callback to handle `Message` objects:

```cpp
class Message {
public:
//...
	bool token_is_valid() const;
	long get_timestamp() const;
	std::wstring get_channel() const;
	std::wstring get_team() const;
	std::wstring get_text() const;
	std::wstring get_user() const;
	std::wstring get_trigger_word() const;
private:
//...
};
```

For incoming web traffic (i.e. `GET` requests), you instead handle `WebResponse` object:

```cpp
class WebResponse {
public:
//...
	std::wstring get_content_type() const;
	std::wstring get_content() const;
private:
//...
};
```

The `content_types` is the [MIME Type][15] of the content received by the request.

Feedback
==
Please [post any issues or bugs][14] you find!
