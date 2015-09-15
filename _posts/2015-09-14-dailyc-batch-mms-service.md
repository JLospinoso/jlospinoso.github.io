---
layout: post
title: dailyc - A Batch Multimedia Message Service
image: /images/dailyc.jpg
date: 2015-09-14 16:00
tag: Send multimedia messages using dailyc
categories: [groovy, gorm, java, Spring Boot, Mogreet, software]
---
[1]: https://developer.mogreet.com/
[2]: https://github.com/JLospinoso/dailyc
[3]: http://projects.spring.io/spring-boot/
[4]: https://gradle.org/
[5]: https://grails.github.io/grails-doc/latest/guide/GORM.html
[6]: https://en.wikipedia.org/wiki/Cron
[7]: https://docs.gradle.org/current/userguide/gradle_wrapper.html
[8]: http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html
[9]: https://github.com/mogreet/javaSDK
[10]: https://github.com/JLospinoso/dailyc/blob/master/mogreet/src/test/java/net/lospi/mogreet/MogreetManagerTest.java
[11]: https://developer.mogreet.com/dashboard
[12]: https://github.com/JLospinoso/dailyc/blob/master/dailyc-core/src/main/groovy/net/lospi/dailyc/DailycScheduler.groovy
[13]: https://github.com/JLospinoso/dailyc/blob/master/config.json
[14]: https://www.mysql.com/

[dailyc][2] is a service for sending multimedia messages at regular intervals. You provide dailyc with a folder full of `jpg`s and a list of messages; it will determine which image and message hasn't been sent in a while (if ever) and fire it off to a list of subscribers.

dailyc uses [Mogreet][1] to send messages. Mogreet offers an easy to use REST API; however, their Java SDK, [mercury][9] is a little rough around the edges (i.e. it is not complete). dailyc implements its own [Java wrapper][10]. This wrapper exists as a separate module in the Gradle project and is abstracted from the multimedia messaging service (the `dailyc-core` module).

The service works really well for sending baby pictures to adoring (petulant?) relatives. It is well suited to this task because, well, that's why I implemented it.

# Getting started
First step is to obtain the dailyc source code, which is available on [Github][2]:

	git clone git@github.com:JLospinoso/dailyc

Since dailyc uses the [gradle wrapper][7], you should be able to execute the `gradlew` script (both on Windows and other operating systems). This script will automagically download Gradle for you, then run whatever Gradle task you give it. Note: you must have the [Java SDK][8] installed.

You can verify that dailyc is compiling and that the unit tests are passing by issuing:

	gradlew build

## Mogreet Account
You will need to sign up for a (free) [Mogreet][1] account to be able to send messages. You get a $25 credit (which goes a long way at ~$0.01 per text).

Once you've obtained an account, open up the [Mogreet Dashboard][11] and take note of the `API Credentials` section. You will need to update the following fields in both  `dailyc-core\src\main\resources\application.properties` and `dailyc-core\src\test\resources\application.properties` with the corresponding information in Mogreet Dashboard:

	clientId=1111
	token=deadbeefdeadbeefdeadbeef
	campaignId=999999

Your `campaignId` is the `MMS campaign id` in the Dashboard.

## Configuration
There are a few configuration files you'll need to modify to get dailyc working correctly. First, let's configure how dailyc schedules tasks. There are three tasks that execute at regular interval [when dailyc is running][12]:

* the batch service, which checks for new messages, subscribers, and images to load into the database,
* the send service, which sends an image and a message to all subscribers, and
* the heartbeat service, which logs a message to let you know that dailyc is still running.

In `dailyc-core\src\main\resources\application.properties`, we have three fields that drive all of the scheduling. These are [cron][6] strings. So, for example, if you'd like heartbeats every hour, batch at 0500, and send at 0600, you would have:

	sendCron=0 0 6 * * *
	batchCron=0 0 5 * * *
	heartbeatCron=0 0 * * * *

Next, we must tell dailyc where our images are stored, whom to subscribe to the service, and what messages to send. All of this information is given in a JSON file pointed to by `application.properties`. Suppose we specify

	configPath=c:/dailyc/config.json

Use the format given in the [sample configuration][13] to create `c:/dailyc/config.json`:

	{
	  "imageDirectory": "c:/dailyc/img",
	  "subscribers": [
		"5551234567",
		"5551234568",
		"..."
	  ],
	  "messages": [
		"Hello from your favorite granddaughter!",
		"Good morning!",
		"..."
	  ]
	}

All you need to do now is paste a body of `.jpg`s into the `imageDirectory` (in this case `c:/dailyc/img`), and `dailyc` is configured to run.

## Advanced configuration
Since dailyc uses [Spring Boot][3] and [GORM][5], it is very simple to configure a persistent store with dailyc. This will allow all of your state to persist across reboots of the service (which is important if you don't want subscribers to get the same images over and over again.)

This is configured--you guessed it--in the `application.properties`. A [Mysql][14] database, for example, could be configured as follows:

	database.driver=com.mysql.jdbc.Driver
	database.url=jdbc:mysql://mysql.domain.com/dailyc
	database.username=dailyc
	database.password=abc123
	
Assuming you've created the database and user correctly, it really is that simple.

# Running dailyc
As dailyc is a [Spring Boot][3] application, execution is as simple as 

	gradlew bootRun
	
Please provide feedback/bug reports for v0.1.0 [on Github][2].