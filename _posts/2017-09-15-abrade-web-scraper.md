---
layout: post
title: Abrade, a high-throughput web API scraper
image: /images/Abrade.svg
date: 2017-09-15 08:00
tag: A TLS- and SOCKS-5-friendly web-resource scraper built on asyncronous network I/O.
categories: [cpp, developing, software]
---
[1]: https://github.com/JLospinoso/abrade
[2]: https://www.openssl.org/
[3]: http://www.boost.org/
[4]: https://hub.docker.com/r/jlospinoso/abrade/
[5]: https://quay.io/repository/jlospinoso/abrade
[6]: https://www.docker.com/
[7]: https://slproweb.com/products/Win32OpenSSL.html
[8]: https://www.visualstudio.com/downloads/
[9]: https://ring.com/
[10]: https://www.torproject.org/download/download.html.en
[11]: https://ifconfig.me
[12]: https://na.leagueoflegends.com
[13]: https://matchhistory.na.leagueoflegends.com/en/#match-details/NA1/2566000059?tab=overview
[14]: https://developer.chrome.com/devtools
[15]: https://gitlab.com
[16]: https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/
[17]: http://manpages.ubuntu.com/manpages/zesty/man5/systemd.unit.5.html
[18]: https://aws.amazon.com/s3/
[19]: https://www.ipify.org/
[20]: https://en.wikipedia.org/wiki/Time_series
[21]: https://en.wikipedia.org/wiki/Stochastic_approximation
[22]: https://developer.telerik.com/featured/front-end-driven-applications-new-approach-applications/
[23]: https://finance.yahoo.com
[24]: https://github.com/jlospinoso/abrade/issues
[25]: https://pastebin.com
[26]: https://www.telerik.com/fiddler
[27]: https://jlospinoso.github.io/software/wireshark/networks/ubuntu/2015/02/11/configuring-wireshark-on-ubuntu-14.html
[28]: https://httpbin.org
[29]: https://www.eff.org/issues/coders
[30]: https://supporters.eff.org/donate

[Abrade][1] is an open-source, command-line tool for collecting web-resources from URLs containing sequential, alpha-numerical IDs. It uses asynchronous network I/O to minimize CPU utilization while maximizing throughput. Abrade has support for TLS/SSL and SOCKS 5 proxies (e.g. TOR), and Abrade works on operating systems where [OpenSSL][2] and [Boost Libraries installed][3] are available (including all major, modern OSs). It's also available as a [Docker][6] container.

_Disclaimer: Make sure that you are not running afoul of applicable laws and regulations and that you obtain permission where necessary when accessing web resources. All of the examples in this post use https://httpbin.org or https://ipify.org which, as of the time of writing, complies with thoes sites' terms of use. Any discussion or mention of other URL patterns is not an endorsement by the author for the use of Abrade to scrape such URLs._

Abrade is really good at probing web APIs like the *Ring Video Doorbell's Shared Videos*. With a one-liner, you could theoretically scrape such an API:

```
> abrade ring.com /share/{1000000000:9999999999} -tf
```

In this blog post, we'll get Abrade up and running in your local environment, then introduce the Abrade's major features. We then proceed with several example illustrating how one might use Abrade to scrape web APIs to include the Ring Video Doorbell, League of Legends game histories, and Yahoo! Finance stock quotes. We'll also touch on running Abrade through a SOCKS 5 proxy like Tor, and how to wire up an Ubuntu instance to host a long-running scrape.

# Up and Running

First, let's get Abrade running in your environment. Just skip to the relevant section for your particular setup. Note that Docker works on all major operating systems, and it is the easiest way to get up and running. If you have no interest in Docker, skip ahead to Windows or Linux as appropriate.

## Docker

Pull the image from [Docker Hub][4]:

```
docker pull jlospinoso/abrade:v0.1.0
```

Or from [quay.io][5]:

```
docker pull quay.io/jlospinoso/abrade:v0.1.0
```

To run Abrade, simply use `docker run`, e.g.:

```
docker run -it --rm quay.io/jlospinoso/abrade:v0.1.0 --help
```

## Windows

1. Download the latest version of abrade.exe from [GitHub][1]. Abrade only supports x64 architectures.
2. Make sure you've got OpenSSL installed. One option is to install [Shining Light Production's build][7]. Select the latest x64 architecture.
3. Install the [Visual Studio 2017 Redistributable][8].
4. In the command line navigate to the directory containing abrade.exe and execute it.

```
> abrade  --help
```

## Linux

1. Download the latest version of abrade from [GitHub][1]. Abrade only supports x64 architectures.
2. Make sure you've got OpenSSL installed. You can install from source fairly easily:
```
git clone https://github.com/openssl/openssl.git
cd openssl
./config
make
sudo make install
```
3. Run abrade:
```
> abrade --help
```

## If you've got the environment set up correctly...

You should be greeted with a friendly help message once you've successfully invoked Abrade:
```
Usage: abrade host pattern:
  --host arg                            host name (eg example.com)
  --pattern arg (=/)                    format of URL (eg ?mynum={1:5}&myhex=0x
                                        {hhhh}). See documentation for
                                        formatting of patterns.
  --agent arg (=Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:47.0) Gecko/20100101 Firefox/47.0)
                                        User-agent string (default: Firefox 47)
  --out arg                             output path. dir if contents enabled.
                                        (default: HOSTNAME)
  --err arg                             error path (file). (default:
                                        HOSTNAME-err.log)
  --proxy arg                           SOCKS5 proxy address:port. (default:
                                        none)
  -t [ --tls ]                          use tls/ssl (default: no)
  -s [ --sensitive ]                    complain about rude TCP teardowns
                                        (default: no)
  -o [ --tor ]                          use local proxy at 127.0.0.1:9050
                                        (default: no)
  -r [ --verify ]                       verify ssl (default: no)
  -l [ --leadzero ]                     output leading zeros in URL (default:
                                        no)
  -e [ --telescoping ]                  telescope patterns (default: no)
  -f [ --found ]                        print when resource found (default:
                                        no). implied by verbose
  -v [ --verbose ]                      prints gratuitous output to console
                                        (default: no)
  -c [ --contents ]                     read full contents (default: no)
  --test                                no network requests, just write
                                        generated URIs to console (default: no)
  -p [ --optimize ]                     Optimize number of simultaneous
                                        requests (default: no)
  -i [ --init ] arg (=1000)             Initial number of simultaneous requests
  --min arg (=1)                        Minimum number of simultaneous requests
  --max arg (=25000)                    Maximum number of simultaneous requests
  --ssize arg (=50)                     Size of velocity sliding window
  --sint arg (=1000)                    Size of sampling interval
  -h [ --help ]                         produce help message
```

# Abrade Patterns

Unlike other web scraping tools that support rendering and crawling of webpages for buried information, Abrade scrapes web URLs directly. In other words, it is great for web APIs.

With the [ubiquity of front-end javascript clients][22] on the web, sites are more frequently exposing web APIs directly to the user's browser. If you open a browser's debugging tools (just press F12), you can monitor the network traffic that your browser is generating at the request of the Javascript running on the page. You can identify endpoints of interest by looking for URLs that appear to have IDs embedded within the URL or GET parameters. Telerik's [Fiddler][26] tool and [Wireshark][27] are also excellent options for inspecting HTTP requests on the wire.

Abrade's niche is to make hundreds or thousands of requests per second to query such embedded URLs. It can be used to query the existence of these URLs or even to save off the contents returned by the server for each valid URL. To tell Abrade how to generate the URLs you want to try, you use _patterns_.

Abrade expects two arguments for a scrape: the _host_ and a _pattern_. The host is either an IP address or a hostname used in a DNS query (omit the HTTP/HTTPS prefix). Patterns come in two forms: _explicit_ and _implicit_. All together, an Abrade invocation looks like this:

```
> abrade httpbin.org anything/my/100/resource?p1=123 --tls
```

This will generate a single HEAD request to https://httpbin.org/anything/my/100/resource?p1=123. Of course, this isn't very interesting from a scraping perspective--it's just a single network request. Where Abrade shines is in generating large volumes of network requests based on patterns.

Throughout the following examples, we will use the excellent site [httpbin.org][28]. Note that you must use `--tls` (or the shorthand version `-t`) if you wish to to generate URLs that you can visit e.g. with a browser, since httpbin does not support unsecured connections.

## Explicit Patterns

Explicit patterns are numerical only, and they contain a START value and an END value (which must be unsigned integers). They have the form:

```
{START:END}
```

Say we want to query URLs of the following form:

```
https://httpbin.org/anything/1
https://httpbin.org/anything/2
https://httpbin.org/anything/3
...
https://httpbin.org/anything/10
```

Given the host `httpbin.org`, the corresponding pattern that will generate these URLs is `/anything/{1:10}`.

It is possible to test a pattern in Abrade using the `--test` option:

```
> abrade httpbin.org /anything/{1:10} --test --tls
[ ] Host: httpbin.org
[ ] Pattern: /anything/{1:10}
# ...
[ ] URL generation set cardinality is 10
[ ] TEST: Writing URIs to console
https://httpbin.org/anything/1
https://httpbin.org/anything/2
https://httpbin.org/anything/3
https://httpbin.org/anything/4
https://httpbin.org/anything/5
https://httpbin.org/anything/6
https://httpbin.org/anything/7
https://httpbin.org/anything/8
https://httpbin.org/anything/9
https://httpbin.org/anything/10
```

Abrade accepts multiple patterns, and will exhaustively enumerate over the implied range of values:

```
> abrade httpbin.org /anything/{1:2}/b/{3:4}?c={1:4} --test --tls
[ ] Host: httpbin.org
[ ] Pattern: /anything/{1:2}/b/{3:4}?c={1:4}
# ...
[ ] TEST: Writing URIs to console
https://httpbin.org/anything/1/b/3?c=1
https://httpbin.org/anything/1/b/3?c=2
https://httpbin.org/anything/1/b/3?c=3
https://httpbin.org/anything/1/b/3?c=4
https://httpbin.org/anything/1/b/4?c=1
https://httpbin.org/anything/1/b/4?c=2
https://httpbin.org/anything/1/b/4?c=3
https://httpbin.org/anything/1/b/4?c=4
https://httpbin.org/anything/2/b/3?c=1
https://httpbin.org/anything/2/b/3?c=2
https://httpbin.org/anything/2/b/3?c=3
https://httpbin.org/anything/2/b/3?c=4
https://httpbin.org/anything/2/b/4?c=1
https://httpbin.org/anything/2/b/4?c=2
https://httpbin.org/anything/2/b/4?c=3
https://httpbin.org/anything/2/b/4?c=4
```

# Implicit Patterns

Sometimes we have more complicated patterns to build. For this, we can use _implicit patterns_ which contain one or more of the following characters:

Char | Description                 | Set
---- |:--------------------------- | ------------------------------------------------------------------
`o`  | An octal                    | `01234567`
`d`  | A digit                     | `0123456789`
`h`  | A lower-case hexadecimal    | `0123456789abcdef`
`H`  | An upper-case hexadecimal   | `0123456789ABCDEF`
`a`  | A lower-case alpha-numeric  | `abcdefghijklmnopqrstuvwxyz`
`A`  | An upper-case alpha-numeric | `ABCDEFGHIJKLMNOPQRSTUVWXYZ`
`n`  | A lower-case alpha-numeric  | `0123456789abcdefghijklmnopqrstuvwxyz`
`N`  | An upper-case alpha-numeric | `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ`
`b`  | An alphanumeric             | `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`

Simply place the desired sequence in braces `{ }`. For example, the pattern `{Ha}` generates an upper-case hexadecimal followed by a lower-case alpha-numeric:

```
> abrade httpbin.org /get?id={Ha} --test --tls
] Host: httpbin.org
[ ] Pattern: /get?id={Ha}
[ ] Include leading zeros: No
[ ] Telescope pattern: No
# ...
[ ] URL generation set cardinality is 416
[ ] TEST: Writing URIs to console
https://httpbin.org/get?id=a
https://httpbin.org/get?id=b
https://httpbin.org/get?id=c
https://httpbin.org/get?id=d
https://httpbin.org/get?id=e
https://httpbin.org/get?id=f
https://httpbin.org/get?id=g
# ...
https://httpbin.org/get?id=x
https://httpbin.org/get?id=y
https://httpbin.org/get?id=z
https://httpbin.org/get?id=1a
https://httpbin.org/get?id=1b
https://httpbin.org/get?id=1c
# ...
https://httpbin.org/get?id=1x
https://httpbin.org/get?id=1y
https://httpbin.org/get?id=1z
https://httpbin.org/get?id=2a
https://httpbin.org/get?id=2b
https://httpbin.org/get?id=2c
# ...
https://httpbin.org/get?id=Fx
https://httpbin.org/get?id=Fy
https://httpbin.org/get?id=Fz
```

Whether you need to telescope your patterns or not depends on the way that your scraping target is generating routes. We'll use this feature later to generate stock ticker symbols (which can range in length from 1 to 5).

### Leading zeros

By default, abrade will omit "leading zeros." This means the first character of whatever set you've chosen (octal, decimal, alphanumeric, etc.) will be omitted unless you provide the `--leadzero` option. In the following example, we've added `--leadzero` and you can see the that they are added to the test output:

```
> abrade httpbin.org /get?id={Ha} --test --leadzero
] Host: httpbin.org
[ ] Pattern: /get?id={Ha}
[ ] Include leading zeros: No
[ ] Telescope pattern: No
# ...
[ ] URL generation set cardinality is 416
[ ] TEST: Writing URIs to console
https://httpbin.org/get?id=0a
https://httpbin.org/get?id=0b
https://httpbin.org/get?id=0c
https://httpbin.org/get?id=0d
https://httpbin.org/get?id=0e
https://httpbin.org/get?id=0f
https://httpbin.org/get?id=0g
# ...
https://httpbin.org/get?id=0x
https://httpbin.org/get?id=0y
https://httpbin.org/get?id=0z
https://httpbin.org/get?id=1a
https://httpbin.org/get?id=1b
https://httpbin.org/get?id=1c
# ...
https://httpbin.org/get?id=1x
https://httpbin.org/get?id=1y
https://httpbin.org/get?id=1z
https://httpbin.org/get?id=2a
https://httpbin.org/get?id=2b
https://httpbin.org/get?id=2c
# ...
https://httpbin.org/get?id=Fx
https://httpbin.org/get?id=Fy
https://httpbin.org/get?id=Fz
```

### Telescoping Patterns

Sometimes, we would also like to include lesser subsets of a given pattern in the results. For example, suppose we have the pattern `{dda}`, but we also want to include `{da}` and `{a}`. We call this a _telescoping_ pattern and we can tell Abrade to telescope patterns by using the `--telescoping` option:

```
> abrade httpbin.org /get?id={dda} --test --leadzero --telescoping --tls
[ ] Host: httpbin.org
[ ] Pattern: /get?id={dda}
[ ] Include leading zeros: Yes
[ ] Telescope pattern: Yes
[ ] TLS/SSL: No
[ ] TLS/SSL Peer Verify: No
[ ] User Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:47.0) Gecko/20100101 Firefox/47.0
[ ] Proxy: No
[ ] Contents: No
# ...
[ ] URL generation set cardinality is 2886
[ ] TEST: Writing URIs to console
https://httpbin.org/get?id=a
https://httpbin.org/get?id=b
https://httpbin.org/get?id=c
# ...
https://httpbin.org/get?id=x
https://httpbin.org/get?id=y
https://httpbin.org/get?id=z
https://httpbin.org/get?id=0a
https://httpbin.org/get?id=0b
https://httpbin.org/get?id=0c
# ...
https://httpbin.org/get?id=9x
https://httpbin.org/get?id=9y
https://httpbin.org/get?id=9z
https://httpbin.org/get?id=00a
https://httpbin.org/get?id=00b
https://httpbin.org/get?id=00c
# ...
https://httpbin.org/get?id=99x
https://httpbin.org/get?id=99y
https://httpbin.org/get?id=99z
```

### Continuations

Sometimes, URLs will repeat the same pattern more than once within a pattern. So long as these are continuous, Abrade has an easy way to encode for this: a _continuation pattern_ `{}`. Suppose we'd like to generate URLs that appear as follows:

```
https://httpbin.org/anything/a/?id=a
https://httpbin.org/anything/b/?id=b
https://httpbin.org/anything/c/?id=c
https://httpbin.org/anything/d/?id=d
# ...
```

We can achieve this with the following invocation that uses a continuation:

```
> abrade httpbin.org /anything/{a}?id={} --test --tls
[ ] Host: httpbin.org
[ ] Pattern: /{a}?id={}
[ ] Include leading zeros: No
[ ] Telescope pattern: No
# ...
[ ] URL generation set cardinality is 26
[ ] TEST: Writing URIs to console
https://httpbin.org/anything/a?id=a
https://httpbin.org/anything/b?id=b
https://httpbin.org/anything/c?id=c
https://httpbin.org/anything/d?id=d
# ...
https://httpbin.org/anything/x?id=x
https://httpbin.org/anything/y?id=y
https://httpbin.org/anything/z?id=z
```

A continuation will mirror the pattern immediately to the left. Placing a continuation as the first pattern will generate an error.

## Cardinality

Once Abrade has parsed all patterns, it will compute the _cardinality_ of the URL space. This is the number of URLs that Abrade will iterate through. At some (very large) number, the cardinality cannot be stored in an integer on your platform. At this point, Abrade will compute the natural log of the cardinality.

In the previous example, the cardinality is 26 + 10*26 + 10*10*26 = 2886.

In the following pattern containing 10 `b` elements, the cardinality is considerably larger:

```
> abrade httpbin.org /anything?id={bbbbbbbbbb} --test --tls
[ ] Host: httpbin.org
[ ] Pattern: /anything?id={bbbbbbbbbb}
# ...
[ ] URL generation set cardinality is 839299365868340224
[ ] TEST: Writing URIs to console
https://httpbin.org/anything/?id=0
https://httpbin.org/anything/?id=1
https://httpbin.org/anything/?id=2
# ...
```

There are 62 characters in `b` (a-z, A-Z, 0-9), so the cardinality of this pattern is 62 ^ 10 = 839299365868340224.

On my x64 machine, the largest value of a "size type" is 2^64-1 = 18446744073709551615. When we add another `b` to the above pattern, we get a cardinality of 21821783512576845824. This is larger than the largest value of a size type, so Abrade computes the cardinality as a natural logarithm:

```
> abrade httpbin.org /anything?id={bbbbbbbbbbb} --test --tls
[ ] Host: httpbin.org
[ ] Pattern: /?id={bbbbbbbbbbbb}
# ...
[!] URL generation set log cardinality is 45.3985
[ ] TEST: Writing URIs to console
https://httpbin.org/anything/?id=0
https://httpbin.org/anything/?id=1
https://httpbin.org/anything/?id=2
# ...
```

It is unlikely that you would be able to perform this many requests (you would need to perform about 2.2 x 10^19 requests per second to get this work done in 100 years), but it's good to know what to look out for. If Abrade computes the cardinality of your pattern as a natural log, don't expect it to complete.

This may be OK--maybe you're just looking for a few hits rather than an exhaustive search.

# Abrading by Example

## Example 1: Scraping the Ring Video Doorbell

![Ring Video Doorbell](https://raw.githubusercontent.com/JLospinoso/jlospinoso.github.io/master/images/ring_doorbell.jpg)

The [Ring Video Doorbell][9] is an internet-connected, video-and-voice-enabled doorbell. It has motion-detection capability in addition to the ability to answer your door remotely. The videos are stored remotely in Ring's servers (or, more accurately, [Amazon S3 buckets][18]). Users can elect to make their videos public by clicking a button in the phone app or using the web portal. This yields a URL with the following format:

```
https://ring.com/share/2001051206
```

The number embedded in this URL is a so-called ding ID. It is sequential and probably corresponds with a unique event id that gets generated each time a doorbell is rung or motion is detected. If the user makes such a video public, you can access the video so long as you have the correct ding ID. There is no authentication, and there no way for a user to revoke access. I submitted a feature request in ticket #2908404 with Ring Support on July 6, 2017.

Say we want to discover public Ring videos with Ding IDs from 4,000,010,000 to 4,000,020,000. We can use an explicit pattern for this:

```
> abrade ring.com /share/{4000000000:4000010000} --tls --found
```

There are two options in use above:

Option     | Description
---------- |:-------------------------------------------
`--tls`    | Use SSL/TLS
`--found`  | Report successfully found resources


_Note: You can use the `--verify` option to force OpenSSL to perform peer verification. This implies the `--tls` option when used._

You can also use the shorthand notation for the `tls` and `found` options, `t` and `f` respectively:

```
> abrade ring.com /share/{4000000000:4000010000} -tf
```

Either of the above invocations of abrade will generate 10001 HEAD requests over TLS/SSL and it will report immediately whenever a valid resource is found:

```
/share/4000010000
/share/4000010001
/share/4000010002
...
...
/share/4000019999
/share/4000020000
```

An example Abrade invocation to scrape Ring Video Doorbells might look like the following:

```
> abrade ring.com /share/{4000000000:4000010000} -tf
[ ] Host: ring.com
[ ] Pattern: /share/{4000000000:4000010000}
[ ] Include leading zeros: No
[ ] Telescope pattern: No
[ ] TLS/SSL: Yes
# ...
```

For HEAD requests, Abrade will output all of the resources it finds (i.e. any response with a 200-series code) to a file corresponding to the hostname. In this case, the file `ring.com` will contain any results from the scrape.

If Abrade runs into any exceptional conditions (e.g. broken network pipe), it will write the error messages out to the file indicated by the `--err` argument, which defaults to `HOSTNAME-err` by default.

We override the default output paths using the following options:

Option  | Description
------- |:--------------------------------------------
`--out` | Modifies the output path for successful URLs
`--err` | Modifies the output path for the error log

For example, this invocation writes successfully found dings to `dings.txt` and error statuses to `my_err.log`:

```
> abrade ring.com /share/{4000000000:4000010000} --tls --found --out=dings.txt --err=my_err.log
```

## Example 2: Using Tor

![Tor](https://raw.githubusercontent.com/JLospinoso/jlospinoso.github.io/master/images/tor.png)

Abrade has basic SOCKS 5 proxy support built-in (it only supports no-authentication mode). One common use-case for this functionality is to use a locally running Tor service to route your requests. You'll need the "Expert Bundle" (Windows) or the "Standalone Tor" (Linux) from [torproject.org][10]. If you run the Tor service with default settings, it will listen for SOCKS 5 connection requests on 127.0.0.1:9050. You can instruct Tor to use such a service using the `--tor` option.

You can determine the IP address you're coming from using [ipify][11]. By visiting [http://api.ipify.org][19], you'll get back a plaintext response containing your IP. We can use [api.ipify.org][19] to check out what IP we're coming from, then compare where we're coming from while using a Tor proxy.

There are two options that will help us in this situation:

* `--verbose` or `v` tells Abrade to print a whole bunch of information (including all requests/responses) to console. Abrade will still write output to disk per usual.
* `--contents` or `c` tells Abrade to perform a GET request rather than a HEAD request. This means the server will respond with the contents of the URL (if it exists).

First, make sure Tor is running locally on your system (usually, it will bind to 127.0.0.1:9050).

Next, invoke Abrade:

```
> abrade api.ipify.org --tls --verbose --contents
```

or

```
> abrade api.ipify.org -tvc
```

Here's a sample response:

```
[ ] Host: api.ipify.org
[ ] Pattern: /
[ ] Proxy: No
[ ] Contents: Yes
[ ] Verbose: Yes
[ ] Print found: Yes
# ...
[ ] URL generation set cardinality is 1
[ ] Payload for /: GET / HTTP/1.1
Host: api.ipify.org
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:47.0) Gecko/20100101 Firefox/47.0

[ ] Response from /:
HTTP/1.1 200 OK
Server: Cowboy
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Type: text/plain
Date: Tue, 22 Aug 2017 12:25:01 GMT
Content-Length: 12
Via: 1.1 vegur

65.55.42.183
[+] Status of /: 200
```

The response tells us that the IP we're coming from is 65.55.42.183 (Redmond, Washington).

Next, let's route the request through Tor. All that's required is to add `--tor` or `-o` to the invocation:

```
> abrade api.ipify.org -tvco
```

I got the following response:

```
[ ] Host: api.ipify.org
[ ] Pattern: /
[ ] Include leading zeros: No
[ ] Telescope pattern: No
[ ] TLS/SSL: Yes
[ ] Verify: No
[ ] Proxy: 127.0.0.1:9050
# ...
[ ] URL generation set cardinality is 1
[ ] Payload for /: GET / HTTP/1.1
Host: api.ipify.org
User-Agent: Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:47.0) Gecko/20100101 Firefox/47.0

[ ] Response from /:
HTTP/1.1 200 OK
Server: Cowboy
Connection: keep-alive
Access-Control-Allow-Origin: *
Content-Type: text/plain
Date: Tue, 22 Aug 2017 12:28:36 GMT
Content-Length: 14
Via: 1.1 vegur

79.172.193.32
[+] Status of /: 200
```

The IP we're coming from is 79.172.193.32 (a Tor exit node in Budapest).

_Note: using Tor can have a severe impact on the performance of Abrade. You are making a tradeoff between anonymity and throughput._

Tor is just another SOCKS 5 proxy server from Abrade's perspective. You can use whatever SOCKS 5 proxy you'd like. In fact, the `--tor` option is just shorthand for `--proxy=127.0.0.1:9050`. Just use the `--proxy` option for more flexibility. Note that since SOCKS 5 proxies support DNS, Abrade will also route your DNS queries through the proxy.

# Example 3: Scraping League of Legends Game Histories

![League of Legends](https://raw.githubusercontent.com/JLospinoso/jlospinoso.github.io/master/images/league_of_legends.png)

[League of Legends][12] is an extremely popular video game. It's a so-called multiplayer online battle arena, where players control champions that work together to destroy the opposing team's base. At any given moment, millions of people are playing around the globe.

After each game (which can last from 20 to 60 minutes or more), the players can view a summary of the game in fairly granular detail. [Here's an example.][13]

Poking around a little bit using the [browser developer tools][14], you can see all kinds of asyncronous requests for data blobs that fuel the views. One such example is the "timeline" resource, which contains a whole host of information about events that occurred throughout the game, including items that players bought, kills and assists, objectives taken, level ups, and gold collected:

```
https://acs.leagueoflegends.com/v1/stats/game/NA1/2566000029/timeline
```

As you might have guessed, the Game ID (like 2566000029) is sequential and numerical, which makes it a prime candidate for Abrade scraping. The following Abrade invocation will do the trick:

```
abrade acs.leagueoflegends.com /v1/stats/game/NA1/{2566000000:2566010000}/timeline --tls --contents
```

(alternately, with abreviated options):

```
abrade acs.leagueoflegends.com /v1/stats/game/NA1/{2566000000:2566010000}/timeline -tc
```

The following is an example of output you might expect:

```
[ ] Host: acs.leagueoflegends.com
[ ] Pattern: /v1/stats/game/NA1/{2566000000:2566010000}/timeline
[ ] Include leading zeros: No
[ ] Telescope pattern: No
[ ] TLS/SSL: Yes
# ...
```

It turns out that not all of the numbers correspond with valid Game IDs. This is probably to do with the region (NA1 for North America 1) embedded in the URL route. Abrade deals with this kind of situation gracefully. All the 2XX-response statuses get written to the acs.leagueoflegends.com-contents folder for further processing. Each of the files contains the full body of the HTTP response.

# Example 4: Yahoo! Finance

[Yahoo! Finance][23] is a free service that provides a wealth of financial information, including articles and stock quotes. Let's take a look at what kinds of network traffic our browser generates when we look at General Electric's stock information:

```
https://finance.yahoo.com/quote/GE
```

It looks like an Apache Spark server is dishing out stock information at this URL:

```
https://query1.finance.yahoo.com/v7/finance/spark?symbols=ES%3DF&range=1d&interval=5m&indicators=close&includeTimestamps=false&includePrePost=false&corsDomain=finance.yahoo.com&.tsrc=finance
```

We can pare this URL down quite a bit using the GET parameter's keys as a guide. What if we want to grab Apple's stock prices (AAPL)?

```
https://query1.finance.yahoo.com/v7/finance/spark?symbols=AAPL
```

Let's say we want to scrape all the prices for all stock tickers. Stock tickers are usually 5 upper-case letters or less. Since we want to scrape all the stock tickers from `A` to `ZZZZZ`, we've got a _telescoping_ implicit pattern:

```
> abrade query1.finance.yahoo.com /v7/finance/spark?symbols={AAAAA} --tls --contents --telescoping
```

Since this is a content scrape, each successful response gets its own file within the output folder. For whatever reason, Yahoo! Finance might return a 200 for _all_ queries, but plops an error code into the JSON payload in the response. It's easy enough to do some post-processing and filter these out. Abrade's job is pulling the payloads down from Yahoo! Finance as quickly as possible.

# Performance Tuning

Abrade is built with raw throughput in mind. By using the asynchronous APIs of the underlying operating system, it can have thousands of requests in flight at any given moment. How many optimize network throughput is an empirical question. Abrade will start out with (up to) 1000 simultaneous requests. You can modify this number with the `--init` argument, which must be greater than zero.

By default, Abrade will use a heuristic algorithm to adapt the number of concurrent network connections. It keeps track of the (a) the average request velocity and (b) the number of concurrent network connections every so often to form a [time series][20]. The number of responses received before an element is added to the time series is given by the `--ssint` argument, which defaults to 1000. The maximum size of the time series is given by the `--ssize` argument, which defaults to 50.

Each time a new sample is taken, Abrade fits an ordinary least squares regression to the time series (velocity as the response variable, number of concurrent connections as the treatment variable). The resulting coefficient is a rough estimate of how an increase in the number of concurrent connections would affect the request velocity. If the coefficient is positive, Abrade increases the number of simultaneous requests by one. If it is negative, Abrade decreases by one instead. This scheme is inspired by the [Robbins-Monro Stochastic Approximation Algorithm][21].

If you have a long-running scraping project, you can fix the number of simultaneous requests (disabling the optimization heuristic) by using the `--fixed` option. This will cause Abrade to maintain exactly the number of simultaneous requests given by the `--init` option.

# Setting up a scraping project on Ubuntu 17.04

You might want to incorporate Abrade into a more robust, long-running scraping campaign. In this section, we'll wrap up with an example weaving together git, systemd, docker, and cron to set up a scraper for Ring videos.

First, set up a git repository to archive the incremental results of your scrapes. This is a simple way to back up your work. I recommend [Gitlab][15] because they have unlimited private repositories. `git clone` this repository into your home directory. In this example, I'll use git@gitlab.com:jalospinoso/wring

Second, make sure that Docker is installed. You can see the latest installation instructions [here][16]. Here's the abridged version:

```
sudo apt-get remove docker docker-engine docker.io
sudo apt-get update
sudo apt-get install \
  linux-image-extra-$(uname -r) \
  linux-image-extra-virtual
sudo apt-get install \
  apt-transport-https \
  ca-certificates \
  curl \
  software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce
sudo docker run hello-world
```

Third, add the following scripts to your home directory:

```
# update_images.sh
docker pull quay.io/jlospinoso/abrade >> /home/jalospinoso/update_images.log
```

```
# update_repos.sh
git -C /home/jalospinoso/wring commit -am "Automated update" >> /home/jalospinoso/update_repos.log
git -C /home/jalospinoso/wring push >> /home/jalospinoso/update_repos.log
```

You'll want to modify `update_repos.sh` to reflect the directory of your newly created git repository. These scripts make sure that (a) you've always got the latest version of Abrade installed and (b) you have pushed all your hard work to a remote git server somewhere.

Fourth, Add `update_images.sh` and `update_repos.sh` to your crontab:

```
# m h  dom mon dow   command
0 8,21 * * * /home/jalospinoso/update_repos.sh
0 0 * * * /home/jalospinoso/update_images.sh
```

Create a script at `/etc/systemd/system/wring.sh` that looks like the following:

```
/usr/bin/docker run --mount type=bind,src=/home/jalospinoso/wring/abrade-out,dst=/out quay.io/jlospinoso/abrade ring.com /share/{$(tail -n 1 /home/jalospinoso/wring/abrade-out/ring.txt | grep -Po '\d+'
| cat -):9999999999} --tls --out /out/ring.txt --err /out/ring-err.txt --found --sint=5000 --init 500
```

This script invokes `docker run` and mounts the directory `/home/jalospinoso/wring/abrade-out` into the `/out` folder of the Docker container. We use `quay.io/jlospinoso/abrade` as the container, then pass the positional arguments into abrade. Notice that we've embedded the following little snippet in as the beginning number for the range of URLs to iterate through:

```
$(tail -n 1 /home/jalospinoso/wring/abrade-out/ring.txt | grep -Po '\d+' | cat -)
```
This pulls the latest successfully scraped URL out of the output file located at `/home/jalospinoso/wring/abrade-out/ring.txt`. This gets piped into `grep`, which extracts the Ding ID out of the URL. Doing this ensures that if the machine reboots, Abrade will pick up somewhere nearer to where it left off (after the last successful URL status code).

Finally, we create the `wring` systemd service by adding the unit file `/etc/systemd/system/wring.service`:

```
[Unit]
Description=Ring Video Doorbell Scraper

[Service]
WorkingDirectory=/home/jalospinoso/wring
ExecStart=/bin/bash /etc/systemd/system/wring.sh
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
```

You can read about systemd unit files [here][17].

You can control the newly-minted wring service using `service`:

```
sudo service wring restart
```

You can check out the output the wring service using `journalctl`:

```
journalctl -u wring
```

# Feedback

Please report bugs and suggestions on [Github][24].

# Thanks

![Ring Video Doorbell](https://jlospinoso.github.io/images/eff.png)

Special thanks to the Electronic Frontier Foundation's [Coders' Rights Project][29].

If you find Abrade useful, please consider [donating to the EFF][30] to support their important work protecting digital privacy and free expression.
