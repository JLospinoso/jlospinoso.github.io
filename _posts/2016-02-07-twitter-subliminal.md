---
layout: post
title: Sending Subliminal Messages via Twitter Retweets
image: /images/TwitterSubliminal.png
date: 2016-02-06 22:00
tag: Use cryptographic hashing to send subliminal messages via retweets.
categories: [subliminal-channel, twitter, poco, cryptography, c++, developing, software]
---
[1]: https://github.com/JLospinoso/twitter-subliminal
[2]: https://dev.twitter.com/streaming/public
[3]: https://dev.twitter.com/rest/public
[4]: https://www.emsec.rub.de/media/crypto/attachments/files/2011/03/subliminal_channels.pdf
[5]: http://cs.gmu.edu/~zduric/cs803/Simmons.pdf
[6]: https://twitter.com/subl1minal
[7]: https://en.wikipedia.org/wiki/SHA-1
[8]: https://en.wikipedia.org/wiki/Negative_binomial_distribution
[9]: http://pocoproject.org/
[10]: https://www.gnupg.org/
[11]: http://filebin.ca/
[12]: http://turl.ca/vypavh
[13]: http://martinolivier.com/open/stegoverview.pdf
[14]: http://www.garykessler.net/library/fsc_stego.html
[15]: https://dev.twitter.com/rest/reference/get/application/rate_limit_status
[16]: https://github.com/google/googletest
[17]: http://www.nptechforgood.com/2010/10/31/is-it-better-to-retweet-old-school-style-or-use-use-twitters-retweet-function/
[18]: https://en.wikipedia.org/wiki/Dependency_injection
[19]: https://en.wikipedia.org/wiki/Generator_%28computer_programming%29
[20]: http://en.cppreference.com/w/cpp/thread/future
[21]: https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization
[22]: https://dev.twitter.com/overview/api/twitter-libraries
[23]: https://curl.haxx.se/libcurl/
[24]: http://en.cppreference.com/w/cpp/utility/bitset

[twitter-subliminal][1] is a suite applications for communicating
[subliminal messages][4] over Twitter. Messages are encoded by finding sub-collisions
in the [SHA-1][7] of tweets ids off the [Public Streaming API][2]. These sub-collisions
are retweeted in order. The recipient hashes the Tweet IDs, collecting the message by
concatenating the sub-collisions. The suite is built on top of [Poco Libraries][9]. Source and
binaries (for Linux, OSX, and Windows) are available [here][1].

Vignette
==
Suppose you have some large blob of data to communicate to another party. We can encrypt this
blob e.g. using [GnuPG][10]:

```sh
gpg --output payload.gpg --encrypt --recipient josh@lospi.net cryptonomicon.txt
```

We can upload this large, now-encrypted blob to a site like [filebin.ca][11], yielding a URL
like [turl.ca/vypavh][12]. What we would like to do is communicate this short URL to our
recipient without raising any suspicions. Techniques like [image steganography][13] can embed
lots and lots of data, but it can be detected [pretty easily][14].

Instead, let's use Twitter's
public Streaming API to encode our message using the first few bits of Retweet's ID SHA1.
(We'll work out the details later.)

Encoding the message--turl.ca/vypavh--is easy-peasy:

```sh
$ tse turl.ca/vypavh
Encoding message of size 14 bytes in 8 bit blocks.
Encoding message: turl.ca/vypavh
Encoded 01110100 via retweet of tweet with id = 695456698992062465.
Encoded 01110101 via retweet of tweet with id = 695456862582341632.
Encoded 01110010 via retweet of tweet with id = 695456850012151808.
Encoded 01101100 via retweet of tweet with id = 694041349398642688.
Encoded 00101110 via retweet of tweet with id = 695456840272842752.
Encoded 01100011 via retweet of tweet with id = 695456812276043776.
Encoded 01100001 via retweet of tweet with id = 695456799663599621.
Encoded 00101111 via retweet of tweet with id = 695456866776784896.
Encoded 01110110 via retweet of tweet with id = 695456971634376704.
Encoded 01111001 via retweet of tweet with id = 695456912901586944.
Encoded 01110000 via retweet of tweet with id = 695456900352077825.
Encoded 01100001 via retweet of tweet with id = 695456933893910530.
Encoded 01110110 via retweet of tweet with id = 695457072301756417.
Encoded 01101000 via retweet of tweet with id = 695457013581504514.
Sent 112 bits.
Completed encoding in  96.14 seconds (  1.16 baud).
```

What does this look like on the other end? Check out [@subl1minal][6].
You'll find its just a bunch of re-tweets off the public Streaming API.


How do we recover our message? Pull down the [@subl1minal][6] statuses,
take the SHA1 hashes of each retweeted ID, pop off the first byte from each,
and reassemble your message:

```sh
$ ./tsd
Decoding subliminal message with block size 8 from Twitter.
Successfully decoded message of size 14 bytes.
turl.ca/vypavh
```

The decoding is far faster than the encoding (not that beating 1 baud is hard...)
for reasons we'll dig into over the next few sections.

Under the Hood
==
The essence of the approach is to consider the message to be sent `M` as a large bit array.
We have some stream of data that we can hash to yield a collection `S`. We choose a block size `b` (an important parameter
for performance that we address in the next section). For each element in `S`,
we take the first `b` bits. If those `b` bits match the next `b` bits that we need to send from `M`,
we mark that element from `S` and repeat.

In the context of twitter, the data stream hash is the SHA1 of Tweet IDs from the streaming API,
and marking the element is a retweet.

Using this approach without any optimization yields a
[negative binomial distribution][8] for the number of tweets you'll wait until your n-bit "block"
gets encoded. The probability of success is `1 - 1/2^n`, since the number of possible blocks is `2^n` and the number of failures `r=1`.

Why not pick a really small `n`? Well, you'll be making a whole lot of retweets to get across even small
messages. For a message with `S` bits in it, you'll need `S/n` retweets. It is (relatively) very fast to encode 4-bit blocks, for example--but you'll need 2 tweets per byte of your message!

To help make the decision about blocksize, there's a performance utility `tsp` available in the [twitter-subliminal][1] suite:

```sh
$ ./tsp -h
usage: tsp OPTIONS
Samples Twitter Streaming API and estimates encoding times.

-h, --help                              display this help
-b, --bwconsole                         specify to log console output in one
                                        color
-lLEVEL, --log.level=LEVEL              specify the logging LEVEL
-fPATH, --log.file=PATH                 specify logging output file PATH
-sSECONDS, --sample-time=SECONDS        specify how many SECONDS to sample
                                        from Twitter Streaming API; specify 0
                                        to skip Streaming test
-uTWEETS, --update.interval=TWEETS      during stream test, specify how many
                                        TWEETS to elapse before giving an
                                        update
-oBLOCKS, --blocks.trial=BLOCKS         during encoding test, specify how many
                                        BLOCKS to encode per blocksize
-tBLOCKSIZE, --encoding.test=BLOCKSIZE  add encoding test for BLOCKSIZE;
                                        multiples permitted; omit to skip
                                        encoding test; valid values [1,20]
```

Let's collect 60 seconds worth of Streaming API and see what this would mean
for various selections of block sizes:

```sh
$ tsp -s60
Sampling Twitter Stream for 60 seconds to estimate velocity.
    Sampled    20 tweets in    2.4 seconds.
    Sampled    40 tweets in    3.4 seconds.
...
    Sampled   980 tweets in   59.4 seconds.
Received 985 tweets in 60.32 seconds (16.33 tweets per second).
Estimated encoding times (no caching, i.e. how long to expect for first block):
    1:     16.329 baud (       58785.43   1-bit blocks per hour)
    2:     10.886 baud (       19595.14   2-bit blocks per hour)
    3:      6.998 baud (        8397.92   3-bit blocks per hour)
    4:      4.354 baud (        3919.03   4-bit blocks per hour)
    5:      2.634 baud (        1896.30   5-bit blocks per hour)
    6:      1.555 baud (         933.10   6-bit blocks per hour)
    7:      0.900 baud (         462.88   7-bit blocks per hour)
    8:      0.512 baud (         230.53   8-bit blocks per hour)
    9:      0.288 baud (         115.04   9-bit blocks per hour)
   10:      0.160 baud (          57.46  10-bit blocks per hour)
   11:      0.088 baud (          28.72  11-bit blocks per hour)
   12:      0.048 baud (          14.36  12-bit blocks per hour)
   13:      0.026 baud (           7.18  13-bit blocks per hour)
   14:      0.014 baud (           3.59  14-bit blocks per hour)
   15:      0.007 baud (           1.79  15-bit blocks per hour)
   16:      0.004 baud (           0.90  16-bit blocks per hour)
   17:      0.002 baud (           0.45  17-bit blocks per hour)
   18:      0.001 baud (           0.22  18-bit blocks per hour)
   19:      0.001 baud (           0.11  19-bit blocks per hour)
   20:      0.000 baud (           0.06  20-bit blocks per hour)
```

Seems really slow; however, we can be a little smarter in our implementation.
Rather than throw out all those blocks that don't match our next `n` message bits,
we can store them away in a big map. This way we build up a sort of SHA1 sub-collision
lookup table that can speed up the encoding process considerably.

How much? well, we can test with `tsp`. Let's see how 4-, 8-, and 12-bit encodings fare:

```sh
$ ./tsp -s0 -t4 -t8 -t12
Beginning encoding with block size 4
   Encoded 0000 (  0 of  10 for size 4)
   Encoded 0001 (  1 of  10 for size 4)
   Encoded 0010 (  2 of  10 for size 4)
   Encoded 0011 (  3 of  10 for size 4)
   Encoded 0100 (  4 of  10 for size 4)
   Encoded 0101 (  5 of  10 for size 4)
   Encoded 0110 (  6 of  10 for size 4)
   Encoded 0111 (  7 of  10 for size 4)
   Encoded 1000 (  8 of  10 for size 4)
   Encoded 1001 (  9 of  10 for size 4)
Blocks of 4 bits encoded at 11.71 baud.
Beginning encoding with block size 8
   Encoded 00000000 (  0 of  10 for size 8)
   Encoded 00000001 (  1 of  10 for size 8)
   Encoded 00000010 (  2 of  10 for size 8)
   Encoded 00000011 (  3 of  10 for size 8)
   Encoded 00000100 (  4 of  10 for size 8)
   Encoded 00000101 (  5 of  10 for size 8)
   Encoded 00000110 (  6 of  10 for size 8)
   Encoded 00000111 (  7 of  10 for size 8)
   Encoded 00001000 (  8 of  10 for size 8)
   Encoded 00001001 (  9 of  10 for size 8)
Blocks of 8 bits encoded at  0.93 baud.
Beginning encoding with block size 12
   Encoded 000000000000 (  0 of  10 for size 12)
   Encoded 000000000001 (  1 of  10 for size 12)
   Encoded 000000000010 (  2 of  10 for size 12)
   Encoded 000000000011 (  3 of  10 for size 12)
   Encoded 000000000100 (  4 of  10 for size 12)
   Encoded 000000000101 (  5 of  10 for size 12)
   Encoded 000000000110 (  6 of  10 for size 12)
   Encoded 000000000111 (  7 of  10 for size 12)
   Encoded 000000001000 (  8 of  10 for size 12)
   Encoded 000000001001 (  9 of  10 for size 12)
Blocks of 12 bits encoded at  0.10 baud.
```

`tsp` is simulating the encodings that `tse` does (without actually submitting
retweets). As you can see, the encodings happened faster than `./tsp -s60` predicted
due to the sub-collision lookup table.

Once the message is received, it might be desired to clear the retweets. This can be
done with the `tsr` utility:

```sh
$ ./tsr
Interrogating Twitter for retweets.
Retrieved 12 statuses for deletion.
Deleted tweet with id 695457076433137664.
Deleted tweet with id 695456978743656449.
Deleted tweet with id 695456977208475649.
Deleted tweet with id 695456975740469248.
Deleted tweet with id 695456974209622021.
Deleted tweet with id 695456875630886912.
Deleted tweet with id 695456872757788672.
Deleted tweet with id 695456871285559296.
Deleted tweet with id 695456869570080768.
Deleted tweet with id 695456866369822722.
Deleted tweet with id 695456864809582592.
Deleted tweet with id 695456701336522752.
```

There are [limits to the number of API calls][15] you can make. For diagnosis
purposes (if e.g. you can't get your message to encode with `tse`), you'll want
to check in with `tsl`:

```sh
$ tsl
Determining Twitter application limit status.
Encoder (/statuses/retweet) not mentioned in rate limits.
Decoder (/statuses/user_timeline) not mentioned in rate limits.
Deleter (/statuses/destroy) is not limited. Used 2 of 180 calls within 15 minute window.
Limit Retriever (/application/rate_limit_status) is not limited. Used 1 of 180 calls within 15 minute window.
Verification (/account/verify_credentials) is not limited. Used 0 of 15 calls within 15 minute window.
```

Finally, there is a [Google Test][16] utility `tst` that will confirm that your environment is set up
properly (see the next section for configuring everything):

```sh
$ tst
[==========] Running 26 tests from 8 test cases.
[----------] Global test environment set-up.
[----------] 5 tests from StringBitIteratorTest
[ RUN      ] StringBitIteratorTest.translates_single_character
[       OK ] StringBitIteratorTest.translates_single_character (0 ms)
[ RUN      ] StringBitIteratorTest.translates_two_characters
[       OK ] StringBitIteratorTest.translates_two_characters (0 ms)
[ RUN      ] StringBitIteratorTest.translates_large_string_of_ones
[       OK ] StringBitIteratorTest.translates_large_string_of_ones (31 ms)
...
```

Installing
==
See the README on the Github repo [twitter-subliminal][1].

Some limitations
==
There are a few limitations:

1. Encoding is slow. For even a few dozen bytes, unless you are okay with even more
dozens of retweets. Encoding can, for example, take several hours if you want multi-byte
collisions (e.g. 16-bit blocks)
2. Since [new-style][17] retweets are used, the messages are
subject to corruption if a the original tweet's poster deletes it. This could be
fixed at some point in the future by using old-style retweets and, say, hashing the
message contents rather than the message ID. Since the messages are subject to this
kind of corruption, it will be important to do some kind of validation on the decoded
string.
3. Twitter rate limits the number of API queries you can make per hour. Currently,
the rate is 180 per hour.

Implementation
==
The tools are implemented in modern C++ (i.e. using C++11 and 14 style) and rely only on [Poco][9] libraries. You might be interested in incorporating some of twitter-subliminal into your own
sneaky-ninja project. This is easy to do as all twitter-subliminal library classes are header-only.

If this is your intent, or if you are interested in peeking behind the curtain, here are some key files you could start your journey in:

* `Twitter.h`: This class implements all the specifics of communicating with the Twitter API.
There is [a C++ library available][22], but (1) it didn't implement the Streaming API, (2) the interface is old-style C++ (C++98: no smart-pointers, no RAII idioms or move semantics, etc.), and (3) it uses [libcurl][23] which sports a pretty spartan API which we would have to contend with when
extending for the Streaming portion. Plus, this gave an opportunity to learn more about [Poco][9]!

This class wraps the following four Twitter API endpoints:

```c++
// https://api.twitter.com/1.1/statuses/user_timeline.json
std::string  timelineUserGet(bool includeRetweets,
                          std::string maxId = "",
                          std::string userId = "");
// https://api.twitter.com/1.1/application/rate_limit_status.json
std::string getRateLimitStatus();

// https://api.twitter.com/1.1/statuses/destroy/
std::string statusDestroyById(const std::string& statusId)

// https://stream.twitter.com/1.1/statuses/sample.json
void stream(std::function<void(std::string)> callback);
```

It would be very straightforward to add new endpoints; see the [implementation][1] for examples.

* `TwitterStream.h`: This class is responsible for taking streaming Tweets from `Twitter.h` via a callback in one end and exposing them as an [iterator][19] to clients. It will queue up tweets internally in an attempt to reduce the latency to the client. As an interesting aside, the class uses [futures][20] to manage the `Twitter` callback loop:

```c++
auto streaming_status = stream_future.wait_for(std::chrono::seconds(0));
if(streaming_status == std::future_status::ready) {
    logger.warning("Stream processing future completed; launching a new one.");
    generate_future();
}
```

* `TwitterBlockEncoder.h`: This class uses a `TwitterStream` to retrieve Tweets from the streaming
API.  The `TwitterBlockEncoder` exposes an `encoder` interface:

```c++
Tweet encode(std::bitset<block_size> block);
```

A large `std::unordered_multimap<std::bitset<block_size>, Tweet>` serves as a cache for matching
what comes in from the `Twitter` iterable and what the client is `encode`-ing. As another aside,
this class makes use of modern c++ concurrency primitives, e.g. the `lock_guard` for [RAII-style
mutex acquisition][21].

```c++
std::lock_guard<std::mutex> lock_acquired(lookup_lock);
```

* `TwitterContainer.h`: All runtime configuration is [constructor injected][18]. The easiest way
to get instances of the library classes above is to use a `TwitterContainer`. Of course, you
are absolutely free to roll your own, or incorporate the object construction directly into your
own project.

Finally, a note on the use of `std::bitset`. C++ (and indeed C) does not allow you to manipulate
bits directly--you've got to jump through some hoops. Since we want to be able to handle arbitrary
block sizes, I elected from the beginning to deal with these blocks as [bitsets][24]. On the plus side,
(1) explicit handling of bits in the block became much more straightforward, (2) the `bitset` is a very compact representation, and (3) they are fast to compare and therefore ideal for keys in the `TwitterBlockEncoder`'s `std::unordered_multimap`. On the flip side, their length is templated, i.e. you must know it at compile-time.

This required us to propagate the template parameter through the object tree, and ultimately generate a big switch statement at the root of the application. Ultimately, if this approach were to be integrated into a larger project, it is likely that a block size could be chosen a-priori and obviate the need to create dozens and dozens of versions of our objects. What we end up paying in the applications is some compile time and some bloat in our binaries.

It is worth noting that these disadvantages could have been mitigated by using either a `std::vector<bool>` or a `boost::dynamic_bitset`, but we would have lost some of the advantages mentioned earlier.
