---
layout: post
title: unfurl, An Entropy-Based Link Vulnerability Tool
image: /images/unfurl.svg
date: 2018-02-08 16:00
tag: A Screening Tool for Analyzing Entropy of Link Generation Algorithms
categories: [python, unfurl, abrade, hacking]
---
[1]: https://www.eff.org/issues/coders
[2]: https://jlospinoso.github.io/responsible%20disclosure/abrade/hacking/2017/10/16/ngpvan-email-subscription.html
[3]: https://bitly.com/
[4]: https://en.wikipedia.org/wiki/URL_shortening
[5]: https://nakedsecurity.sophos.com/2014/09/04/5-things-you-should-know-about-email-unsubscribe-links-before-clicking/
[6]: https://mailchimp.com/
[7]: https://github.com/JLospinoso/unfurl
[8]: https://jlospinoso.github.io/cpp/developing/software/2017/09/15/abrade-web-scraper.html
[9]: https://en.wikipedia.org/wiki/Entropy_(information_theory)
[10]: https://raw.githubusercontent.com/JLospinoso/unfurl/master/actmyngpcom.txt
[11]: https://techwelkin.com/understanding-the-components-and-structure-of-a-url
[12]: https://www.grc.com/haystack.htm
[13]: https://en.wikipedia.org/wiki/Probability_mass_function
[14]: https://developers.google.com/gmail/api/quickstart/python
[16]: https://github.com/JLospinoso/unfurl/blob/master/gmail.py
[17]: https://github.com/JLospinoso/unfurl/blob/master/parse_emails.py
[18]: https://github.com/JLospinoso/unfurl/blob/master/unfurl.py

Software engineers use link generation algorithms when they need to provide privileged access to a user, but the user has not prearranged an authentication mechanism like a password. Knowledge of the URL is the secret information that authenticates a user. It's vital that these link generation algorithms have *high entropy* so that an attacker cannot brute force URLs to gain unauthorized access. [unfurl][7] is a tool that analyzes large collections of URLs and estimates their entropies to sift out URLs that might be vulnerable to attack.

In this blog post, we'll demonstrate how to download an entire inbox from [Gmail][8] and extract a large collection of URLs. We'll feed this collection into [unfurl][7] to analyze each link's entropy. Using [unfurl][7], we'll show how to find the [Democratic Donor Database vulnerability] disclosed in November of 2017.

# Link Generation
Link generation algorithms have many uses including in [URL shortening services][4] like [bit.ly][3] and [marketing automation services][5] like [MailChimp][6].

Consider the following bit.ly links to see if you can divine a pattern:

```
https://bit.ly/2nUzGcK
https://bit.ly/2EdO76I
https://bit.ly/2nMUaVA
https://bit.ly/2skwqN8
https://bit.ly/2Ed9lO3
https://bit.ly/2ERnth5
```

Each of these links has a single resource beginning with the number 2. The rest of the resource contains 6 alphanumeric digits.

We could use the [Abrade web scraper][8] to try scraping these URLs with the following invocation:

```
$ abrade --host bit.ly --pattern /2{bbbbbb} --tls --test --leadzero
[ ] Host: bit.ly
[ ] Pattern: /2{bbbbbb}
...
[ ] URL generation set cardinality is 56800235584
[ ] TEST: Writing URIs to console
https://bit.ly/2000000
https://bit.ly/2000001
https://bit.ly/2000002
https://bit.ly/2000003
https://bit.ly/2000004
https://bit.ly/2000005
https://bit.ly/2000006
https://bit.ly/2000007
https://bit.ly/2000008
https://bit.ly/2000009
https://bit.ly/200000A
https://bit.ly/200000B
https://bit.ly/200000C
...
```

Abrade tells us that there are 56,800,235,584--almost 57 billion--elements in the pattern we've specified. How does Abrade perform such a calculation?

An alphanumeric character can take on any value a-z, A-Z, or 0-9, which is 26 + 26 + 10 = 62 possible values. We can compute the number of values the sequence can take (the set
's "cardinality") by multiplying this number--62--by the number of digits in the sequence, 6:

```
62 * 62 * 62 * 62 * 62 * 62 = 62 ^ 6 = 56800235584
```

Now let's consider how we could encode this sequence as a series of bits rather than as six alphanumeric values. Base64 encoding provides a good approximation here, since each Base64 character encodes six bits. This means we could encode six alphanumeric characters in 6 bits * 6 digits = 36 bits.

# Information Entropy
The back-of-the-envelope calculation we've just walked through gives us an intuition for [information entropy][9], the average amount of information produced by a random process.

We can also interpret information entropy (or just "entropy") as answer to the question: how many bits would it take to encode all possible values in a set?

Consider the set of values (A, B, C, D). We could encode this set of four values with two bits:

```
00 -> A
01 -> B
10 -> C
11 -> D
```

When we double the size of the set, we need an extra bit. Consider now the set of values (A, B, C, D, E, F, G, H):

```
000 -> A
001 -> B
010 -> C
011 -> D
100 -> H
101 -> I
110 -> J
111 -> K
```

This illustrates an important point: every bit of entropy doubles the set of value's size.

To compute the exact information entropy `S`, we can take the base-2 log of the set size `N`:

```
S = log_2 N
```

For example, the bit.ly URL link generation has an information entropy of

```
S = log_2 56800235584 = 35.73
```

As expected, this is slightly smaller than our rough estimate of 36 bits.

For a nice discussion of password entropy analysis, see Steve Gibson's [Password Haystacks][12].

# unfurl

[unfurl][7] is a screening tool for automating URL entropy analysis. The big idea is to find tokens in a large list of URLs that have low entropy. These might be susceptible to brute force attacks.

You can obtain roughly 900 sample URLs from the [NGP VAN disclosure][2] from the [unfurl repo][10] https://raw.githubusercontent.com/JLospinoso/unfurl/master/actmyngpcom.txt:

```
https://act.myngp.com/el/-1019536521446290944/6466136548250749440
https://act.myngp.com/el/-1019536521446290944/7763173240933452288
https://act.myngp.com/el/-1019536521446290944/7763173240933452288
https://act.myngp.com/el/-1019536521446290944/7835230834971380224
https://act.myngp.com/el/-1019536521446290944/7907288429009308160?refcode=5092017
https://act.myngp.com/el/-106179900244227584/-6248367425723037184?refcode=thermometer
...
```

unfurl will parse each url into tokens. Each component of the [URL][11] gets parsed, including the resources and the GET parameters. It's OK if the URLs contain different resources, unfurl will differentiate each combination of resources and GET parameters into a separate group for analysis.

Next, unfurl attempts to apply several decoding schemes to each of the encodings:

* Signed integer: treat the token as a signed long integer (8 bytes).
* Unsigned integer: treat the token as an unsigned long integer (8 bytes).
* Hex ASCII: treat the token as hexlified binary (e.g. `0a4bc0ff3e`)
* Base64: treat the token as Base-64-encoded bytes
* ASCII: treat the token as ASCII text

For each successful group-token-decoding combination, unfurl estimates information entropy by computing a [probability mass function][13] (PMF). Each token is taken as a byte array, and for each index `i` we compute the PMF `F_i(X)` of that byte `X` across all tokens in the dataset. This allows us to estimate an information entropy for each index `S_i` in the following way:

```
S_i = -1 * sum_X( F_i(x) * log_2 F_i(x) )
```

We can compute the information entropy of a token by summing `S_i` across all the indices. Using these computations, unfurl produces a report for us describing which decodings produced the lowest-entropy results.

# Using unfurl
[unfurl.py][18] is a command line tool requiring Python 3:

```
> unfurl.py --help
usage: unfurl.py [-h] [-s SEARCH] [-t] [-e ELEM] [-u URL] file

Computes entropy of URLs

positional arguments:
  file                  File containing URL samples

optional arguments:
  -h, --help            show this help message and exit
  -s SEARCH, --search SEARCH
                        Input is a directory. Write multiple outputs to this
                        directory.
  -t, --text            Text output. (Default is JSON.)
  -e ELEM, --elem ELEM  Number of token samples to print (text only)
  -u URL, --url URL     Number of url samples to print (text only)
```

We can execute unfurl on the NGP VAN samples using the following invocation:

```
> unfurl.py actmyngp.com.txt -t -e 2 -o 2
```

The output contains an abundance of information:

```
==== 4  ====
     Sample URLs:
     * https://act.myngp.com/el/-1019536521446290944/6466136548250749440
     * https://act.myngp.com/el/-1019536521446290944/7763173240933452288

     Entropy:  39.1
     **** @2 - hex ****
          Token Entropy: 21.02
          Decode Count:   23
          Sample Tokens:
              166837620140673536
              16 68 37 62 01 40 67 35 36
              h7b@g56

              166837620140673536
              16 68 37 62 01 40 67 35 36
              h7b@g56

     **** @2 - unsigned_int ****
          Token Entropy: 23.32
          Decode Count:   343
          Sample Tokens:
              1074076575825660416
              00 0a a8 7b f4 e2 e7 0e

?{???

              1074076575825660416
              00 0a a8 7b f4 e2 e7 0e

?{???

     **** @2 - signed_int ****
          Token Entropy: 24.97
          Decode Count:   620
          Sample Tokens:
              -1019536521446290944
              00 0a a8 7b ef e0 d9 f1

?{????

              -1019536521446290944
              00 0a a8 7b ef e0 d9 f1

?{????

     **** @3 - unsigned_int ****
          Token Entropy: 18.07
          Decode Count:   310
          Sample Tokens:
              6466136548250749440
              00 0a a8 7b ee 54 bc 59

?{?T?Y

              7763173240933452288
              00 0a a8 7b ee 54 bc 6b

?{?T?k

     **** @3 - signed_int ****
          Token Entropy: 19.28
          Decode Count:   620
          Sample Tokens:
              6466136548250749440
              00 0a a8 7b ee 54 bc 59

?{?T?Y

              7763173240933452288
              00 0a a8 7b ee 54 bc 6b

?{?T?k
...
```

The first line identifies the unique grouping of URL resources and GET parameters. In the snippet above, we have the group name `==== 4  ====`, which corresponds to 4 URL components (domains, resources) and no GET parameters. We can see some sample URLs fitting this description:

```
* https://act.myngp.com/el/-1019536521446290944/6466136548250749440
* https://act.myngp.com/el/-1019536521446290944/7763173240933452288
```

Next, unfurl goes through each token in the URL and presents us with possible decodings and their associated entropies. For the token at index `@2`, hex, signed integer, unsigned integer, and signed integers are all possibilties. The `Decode Count` tells us how many of the URLs can be successfully decoded using each of these algorithms, and the Token Entropy tells us what the estimated information entropy is.

Beneath each result, unfurl lists a few sample tokens, their integer representation, their hexlified representation, and their ASCII representation. As you can see, `signed_int` matches both `@2` and `@3` very well, decoding 620 elements correctly. The associated entropy is 24.97 for `@1` and 19.28 for `@2`. These are low numbers, and would warrant further investigation.

# Trying unfurl Out on a Gmail Inbox

The [unfurl][7] repository also contains an adaptation of the [Gmail Python API][14] in [gmail.py][15], which will download your inbox to a local file. Next, you can use [parse_emails.py][16] to strip the URLs from this file.

# Thanks

Special thanks to the Electronic Frontier Foundation's [Coders' Rights Project][1].
