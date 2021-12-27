---
layout: post
title: Rivestment&#58; A Game of Hash Collisions
image: /images/rivestment.svg
date: 2016-12-18 12:00
tag: Rivestment is a Slack-chat integrated, multiplayer game for aspiring programmers.
categories: [node, javascript, matterbot, slack, c++, mccwa, game, md5, rivestment, docker]
---
[1]: https://github.com/JLospinoso/rivestment
[2]: https://github.com/JLospinoso/rivestment/issues
[3]: https://nodejs.org/en/
[4]: https://www.mongodb.com/
[5]: https://slack.com/
[6]: https://github.com/JLospinoso/matterbot
[7]: https://github.com/JLospinoso/mccwa
[8]: https://about.mattermost.com/
[9]: https://github.com/JLospinoso/rivestment/blob/master/docker-compose.yml-remote
[10]: https://docker.com
[11]: https://slack.com/apps/build
[12]: https://en.wikipedia.org/wiki/MD5
[13]: https://aws.amazon.com/free/

I've been teaching a [C++ course for beginners recently][7]. It's a complicated
language, and sometimes the students can get disinterested pretty quickly. "It's
so hard to do basic stuff," goes a common line of thinking, "why can't I just
use Python or something?" In an attempt to keep students interested in learning
the language, I integrated the [Slack][5] chat framework into the course and
encouraged the students to collaborate there.

[![Modern C++ for the C-Wielding Aspirant](https://jlospinoso.github.io/images/mccwa.svg)][7]

Students love Slack. So I started gearing all of the labs and exercises towards
implementing chat bots using the [Matterbot framework][6] (Matterbot works with
both Slack and the open-source alternative [Mattermost][8]). Learning about
`std::sort`? Make a command that sorts a user's sentence. Trying to grasp
`std::shared_ptr`? Let's pass one out to your commands so that you can
post multiple responses for a given command.

This approach has the major benefit that the students are creating very high-level
functionality (the low-level details are hidden from them within the Matterbot
framework), but they are learning how to express themselves in C++.

Rivestment
==

As a culminating exercise, the students compete with each other to find hash
collisions. They build bots that interact with the score server to automate the
process of requesting challenges and submitting the answers. Because all of the
I/O can be plainly read in Slack, the students are able to iterate quickly and
focus on building a fast, efficient reverse lookup table (or do the collisions
online.)

This game is called [Rivestment][1] (a portmanteau of "Ronald Rivest," the inventor
of the MD5 hashing algorithm, and "investment," because you must spend points
to get new challenges) and I've open-sourced it.

The Competition
==
Each competitor starts with a certain number of points. These points can be cashed
in to the Rivestment server, and the server will respond with some number of
MD5 hashes. The competitor must [find the preimage][12] that generated the MD5s.
Once a solution is found, the competitor submits the preimage/MD5 pair to the
game server and gets points.

The competitor can adjust their level, which will increase the length of the
preimages that they receive (although they are uniformly, randomly distributed).
While finding preimages gets exponentially harder when they are longer, the
Rivestment server gives out more points for longer preimages.

To make the MD5 preimage attack plausible for students running their bots on a
modern laptop, the alphabet/range of possible preimages is severely restricted.
In my experience, three or four letters produces a desirable level of difficulty.

Hosting Rivestment with Docker
==
Starting up a server is really straightforward
thanks to [Docker][10]:

1. Download a `docker-compose.yml` template [from github][9].
2. Create a new [Slack channel][5].
3. Create a [Slack Custom Integration][11] and save the bot's token for later.

Now configure `docker-compose.yml` with your token, e.g.:

```yml
version: '2'

services:
  mongo:
    image: quay.io/jlospinoso/rivest-mongo:v0.2.1
    expose:
    - "27017"
    restart: always
  rivestment:
    image: quay.io/jlospinoso/rivestment:v0.2.1
    ports:
    - "80:80"
    restart: always
    links:
    - mongo:mongo
    entrypoint: npm start
    environment:
    - MONGO_URL=mongodb://mongo:27017/rivestment
    - BOT_NAME=rivestment
    - CHALLENGE_COST=5
    - COLLECTION_NAME=profiles
    - DEBUG_MODE=True
    - DIFFICULTY_RANGE=10
    - INCORRECT_PENALTY=5
    - MAX_SUBMISSIONS=100
    - MAX_LEVEL=25
    - MAX_SCRAPS=350
    - N_CHALLENGES=25
    - PASSWORD_SIZE=6
    - PASSWORD_RANGE=abcdefghijklmnopqrstuvwxyz0123456789
    - PREIMAGE_RANGE=hark
    - SLACK_TOKEN=xoxb-118324377924-cFaZgiH5O2zz82L5wE3E4FS5
    - STARTING_SCORE=1000
```

Save this file on the game host, and issue the command:

```sh
docker-compose up
```

You've got a webpage bound to port `:80` on the host that--through the magic of
websockets--gives real-time updates about the competition to connected web clients.
There's a `Debug` tab that will allow you and the competitors to see the issued challenges
and their correct answers (for help in debugging!) The keys will be omitted from the
`profiles.json` response whenever the `DEBUG_MODE` environment variable is `False`.
(You'll have to restart the `rivestment` Docker image to see the changes).

_Note: In my experience, Rivestment has run very smoothly on a `t2.micro` EC2
image (which, by the way, is [free-tier eligible][13])._

Game Instructions (for Competitors)
==

_When you build and run Rivestment with the compose file above, you'll be able
to navigate to an `Instructions` tab where you'll see the instructions contained here:_

  > The goal of Rivestment is to win.
  >
  > To win, you must have the highest score.
  >
  > You increase your score by finding the preimages that generate MD5 hashes. The competition is set up so that
  > you will benefit greatly from automating submissions using a Slack bot. First, you (or your bot)
  > must register to play in Slack. The user responsible for being referee of the competition is
  > rivestment. To register with rivestment, simply issue the following command:
  >
  > ```
  > rivestment register [NAME]
  > ```
  >
  > where `[NAME]` is the name you'd like rivestment to refer to you as. It must be unique
  > across all players, and you can only register one name at a time. You should probably use the name of your bot here,
  > since you'll want your bot to perform some action for many of rivestment's responses.
  >
  > At any time, you can quit the competition (and even rejoin). Your points will be reset, and you will
  > have the opportunity to change your `[NAME]`. Just issue the following command:
  >
  > ```
  > rivestment quit
  > ```
  >
  > The main interaction between you (or your bot) and rivestment is with the `challenge` and `try` verbs.
  > You start with 1000 points, and you may spend 5
  > per challenge to generate a set of
  > problems. You simply issue the following command:
  >
  > ```
  > rivestment challenge
  > ```
  >
  > and rivestment will respond with a strictly formatted string corresponding to your challenges, e.g.:
  >
  > ```
  > jbot challenges 303d14ff089e96ef12918f26679cebb6 a6ae2e0501b9757668b360d71efe75cf
  > d5777df5589dff6fa803ffdfc2d9434d d129a5ef717217ce56e7f6724842ea1e
  > 9fe198993f1e9ae25ba2e81121bb7ac2 d5f2d6187ab8494115e9607b9eed445e
  > 6b65fdbfa3a1bfbc0e6a3081f6629602 294df8b85ad22f3911ec04c4383cbc1b
  > 7547a67db8f3eddcbf981d7d19018283 2c1dc90e3d591a8fd17b3ed1c941bd86
  > ```
  >
  > _(There are no newlines in the actual responses.)_
  >
  > By default, you will receive 25 challenges. You may request any number of challenges you'd like,
  > but you cannot have more than 350 at any time. To request some other number of challenges, simply
  > append this number to your challenges command, i.e.:
  >
  >
  > ```
  > rivestment challenge 2
  > ```
  >
  > A sample response:
  >
  > ```
  > jbot challenges 303d14ff089e96ef12918f26679cebb6 a6ae2e0501b9757668b360d71efe75cf
  > ```
  >
  >
  > The string is space delimited. The first token is the name that you registered with (*note that this will cause Slack
  > bots to trigger!*). Next is the word "challenges", which you may use in a Slack bot to trigger a command. The remainder of
  > the string is a series of hashes delimited by spaces. The preimage will
  > end with your password, and a random number of salt digits are prepended. The valid alphabet for this salt is
  > the following string: *hark*.
  >
  > Once you have found the preimage for one of your challenges, simply tell rivestment by using the try
  > command, e.g.:
  >
  > ```
  > rivestment try 303d14ff089e96ef12918f26679cebb6 hkhrrav0phm
  > ```
  >
  > You may submit multiple hashes in one `try`. Simply space delimit the hash/preimage pairs and continue adding them
  > to the submission (up to 100).
  >
  > The third token is the hash that you have found, and the fourth token is the corresponding preimage.
  > Note that my password, `av0phm`, is at the end of my preimage. The salt portion for this example is `hkhrr`.
  > If you submit an incorrect hash, you lose 5 points. If you submit a correct answer, you will gain
  > a number of points equal to one plus the length of the salt. This number is also known as the hash's *difficulty*.
  >
  > You may adjust the minimum *difficulty* of the problems that rivestment sends you by issuing the following command:
  >
  > ```
  > rivestment level 5
  > ```
  >
  > In this example, I will no longer receive challenges with preimage salts less than size four. The difficulties of your
  > challenges will range from your `level` to ten plus your `level` (inclusive).
  >
  > You may retrieve your password by issuing the following command:
  >
  > ```
  > rivestment password
  > ```
  >
  > You may retrieve your current score by issuing the following command:
  >
  > ```
  > scorebot points
  > ```
  >
  > You may find that, over time, rivestment will accumulate unsolved hashes called _scraps_. You can find out
  > what these problems are by issuing the following command:
  >
  > ```
  > rivestment scraps
  > ```
  >
  > The response is in an analogous format to the `challenges` response, e.g.
  >
  > ```
  > jbot scraps a6ae2e0501b9757668b360d71efe75cf d5777df5589dff6fa803ffdfc2d9434d
  > d129a5ef717217ce56e7f6724842ea1e 9fe198993f1e9ae25ba2e81121bb7ac2
  > d5f2d6187ab8494115e9607b9eed445e 6b65fdbfa3a1bfbc0e6a3081f6629602
  > 294df8b85ad22f3911ec04c4383cbc1b 7547a67db8f3eddcbf981d7d19018283
  > 2c1dc90e3d591a8fd17b3ed1c941bd86
  > ```
  >
  > _Note that, in this example, the `[NAME]` I registered was `jbot`_
  >
  > If you automate your submissions (you will probably need to do this to be successful), please rate limit your
  > submissions to no more than one per second. Slack will rate limit rivestment and it will halt everyone's
  > ability to interact with rivestment.

A Note on C++
==

While this competition generally rewards students who are able to (a) build a working
bot that communicates with the Rivestment server, then (b) able to squeeze memory and
compute efficiency out of their code, it would be totally possible to run this game
for students learning other languages. Some experimentation would be needed to tune the
parameters of the competition (e.g. by scaling the preimage alphabet, reducing the
`CHALLENGE_COST`, and tightening the `DIFFICULTY_RANGE`).

Feedback
==
Please [post any issues or bugs][2] you find!
