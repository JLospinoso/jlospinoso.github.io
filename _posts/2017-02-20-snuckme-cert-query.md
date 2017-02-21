---
layout: post
title: snuck.me, a service detecting SSL man-in-the-middle
image: /images/snuckme.svg
date: 2017-02-20 12:00
tag: snuck.me allows users to compare legitimate SSL certs with whatever their browser is getting.
categories: [node, javascript, security, cryptography, privacy]
---
[1]: https://snuck.me
[2]: https://www.google.com
[3]: https://snuck.me/tutorial.svg

[snuck.me][1] is a web service for querying an arbitrary site's SSL certificate. A user can compare the results of this query with the certificate that her browser is reporting to help determine of there is a man in the middle:

![snuck.me Infographic](https://snuck.me/tutorial.svg)

# How it works

[snuck.me][1] works by embedding a public key directly into the website's source:

```js
<script src="https://cdnjs.cloudflare.com/ajax/libs/jsencrypt/2.3.1/jsencrypt.min.js" integrity="sha256-WgvkBqG9+UolqdFC1BJOPcy961WTzXj7C9I034ndc4k=" crossorigin="anonymous"></script>
const encrypt = new JSEncrypt();
encrypt.setPublicKey(`-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsvcZU2It/Cjv12DcLUfE
BXrm+DH2v4x+dyH45Rka95JIAMrmu4OdMjSQQbqhb2pYFVOpRfhUoCu50mOKrmGe
f+ILjBnDtpyTpKf+9QsgmVSfeFnlf6Tew0qgKyUiO9E4cmm14BbqjJrYWGR/0Qas
OSRAWX1SoVzho/sSMBwuadekdaC77Pfvk5uMJUkgck5BzQBLCuPXmLsDsNoAmGck
cfTuEF+s2ae+PeHjhH6g2VaIgqVSaOTe3e2O8Dfukw8GQ5q03kmvA5N0sA+9kk07
ntve3xIBZOUpmB7xEHkG8hHjI6j3oVESo2/K764my0F3JZ9iVSH0jnVASQQ0nmAh
HQIDAQAB
-----END PUBLIC KEY-----`);
```

When the user wants to check a certificate, she generates a JSON payload to send to the [snuck.me][1] service. This payload contains the URL corresponding to the certificate that the user wants to retrieve, and a random 20 character password:

```js
const alphabet = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
function randomString(length) {
    let result = '';
    for (let i = length; i > 0; --i) result += alphabet[Math.floor(Math.random() * alphabet.length)];
    return result;
}
let password = randomString(20);
const optionsObj = {
    url: $("#url").val(),
    password: password
};
const options = JSON.stringify(optionsObj);
```

All of this is done in the browser, i.e. on the user's machine. The payload is encrypted using [snuck.me][1]'s public key:

```js
const options = JSON.stringify(optionsObj);
const encryptedEncodedOptions = encrypt.encrypt(options).replace(/\//g, "_").replace(/\+/g, "-");
```

This payload is then sent to the remote [snuck.me][1] server for processing:

```js
// urlPrefix is set to a CORS-enabled Amazon API Gateway URL
const url = urlPrefix + encryptedEncodedOptions;
$.ajax({url: url,
  // ...
);
```

[snuck.me][1] decrypts the payload using its (secret) private key. It queries the requested URL for certificate information, then uses the _user provided_ password to encrypt the certificate using AES-256. This payload is sent back in the body of the HTTP response to the client.

The client simply decrypts the results with the password it provided to [snuck.me][1] and displays the results:

```js
<script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/3.1.9-1/crypto-js.min.js" integrity="sha256-u6BamZiW5tCemje2nrteKC2KoLIKX9lKPSpvCkOhamw=" crossorigin="anonymous"></script>

$.ajax({url: url,
    success: function(response){
        const plainbytes = CryptoJS.AES.decrypt(response, password);
        const plaintext = plainbytes.toString(CryptoJS.enc.Utf8);
        const crt = JSON.parse(plaintext);
        // Render the certificate.
    },
    error: function(result) {
        // Report issues to the user.
    }
});
```

# Why does this work?

This technique works because the man in the middle cannot modify your query. It is encrypted with [snuck.me][1]'s public key, and only [snuck.me][1] can decrypt it. Since the man in the middle cannot know the password you provided in your query, it cannot return bogus results to you by encrypting a spoofed response.

If the man in the middle knows about [snuck.me][1], he could modify [snuck.me][1]'s source. If you want to protect against this attack, you'll need to verify the source out of band. One way to do this is to visit [snuck.me][1] from a known good location and hash the source. You can then hash the source of [snuck.me][1] in the at-risk setting and compare hashes.

Of course, all bets are off if the man in the middle is also a man in your device. You should give up any expectations of privacy in this case.

# Example

We'll wrap this post up with an example where we inspect Google's certificate with [snuck.me][1] using Firefox 51.0.1:

* Visit [https://google.com][2]. You should see the familiar SSL indicator:

![Example 1](https://github.com/JLospinoso/jlospinoso.github.io/raw/master/images/snuckme/img01.PNG)

* Click the lock:

![Example 2](https://github.com/JLospinoso/jlospinoso.github.io/raw/master/images/snuckme/img02.PNG)

* Click the right arrow to see additional information about the certificate:

![Example 3](https://github.com/JLospinoso/jlospinoso.github.io/raw/master/images/snuckme/img03.PNG)

* Click "More Information":

![Example 4](https://github.com/JLospinoso/jlospinoso.github.io/raw/master/images/snuckme/img04.PNG)

* Click on the security tab, then "View Certificate":

![Example 5](https://github.com/JLospinoso/jlospinoso.github.io/raw/master/images/snuckme/img05.PNG)

* Keep this open. Now visit  [snuck.me][1] and query `www.google.com`:

![Example 6](https://github.com/JLospinoso/jlospinoso.github.io/raw/master/images/snuckme/img06.PNG)

Compare the SHA1 Fingerprints of the results! If there's a mismatch, you've probably got a man in the middle.

# Feedback

Please email me (sneakme at lospi dot net) with any bugs!
