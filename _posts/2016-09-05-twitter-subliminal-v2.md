---
layout: post
title: Twitter Subliminal 0.2.0
image: /images/TwitterSubliminal.png
date: 2016-09-05 12:00
tag: Twitter Subliminal Now Dockerized
categories: [subliminal-channel, twitter, poco, cryptography, c++, developing, software]
---
[1]: https://github.com/JLospinoso/twitter-subliminal
[2]: https://docker.com
[3]: https://apps.twitter.com/
[4]: https://jlospinoso.github.io/subliminal-channel/twitter/poco/cryptography/c++/developing/software/2016/02/06/twitter-subliminal.html
[5]: https://github.com/JLospinoso/twitter-subliminal/issues

Getting [twitter-subliminal][1] setup can be a pain, especially in non-Linux
environments. [Docker][2] is a great solution for making software portable, and
in version 0.2, I've Dockerized twitter-subliminal. You still need to signup for
[Twitter API credentials][3], but these are now injectable as command line parameters:

```sh
docker run -it --rm \
     -e consumer.key="ABCD1234" \
     -e secret.key="ABCD1234" \
     -e access.token="ABCD1234-ABCD1234" \
     -e access.token.secret="ABCD1234" \
     jlospinoso/twitter-subliminal
```

By using the `-it` option, you'll get an interactive session with the Docker
container. The working directory will contain all of the usual binaries [available
from v0.1.0][4]:

* tse: message encryption
* tsd: message decryption
* tsr: reset all retweets
* tsp: performance testing for selecting block size
* tst: unit testing
* tsl: check rate limit status with Twitter API

You can also run these binaries directly. To send, e.g.:

```sh
docker run \
     -e consumer.key="ABCD1234" \
     -e secret.key="ABCD1234" \
     -e access.token="ABCD1234-ABCD1234" \
     -e access.token.secret="ABCD1234" \
     jlospinoso/twitter-subliminal ./tse "Hello, world!"
```

There are two locations where the twitter-subliminal containers are published:

* https://hub.docker.com/r/jlospinoso/twitter-subliminal/
* https://quay.io/repository/jlospinoso/twitter-subliminal

Feedback
==
Please [post any issues or bugs][5] you find!
