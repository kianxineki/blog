# offline decentralized single sign-on in the browser

Recently, browsers have just begun to implement
[web cryptography](http://www.w3.org/TR/WebCryptoAPI/).
This means that browsers are now capable of the same kind of passwordless
decentralized authentication schemes we've had server-side with ssh and tls
asymmetric keys for decades.

Imagine if you could just generate a key and sign messages with that key,
proving your identity to other users and backend services without the need for a
password or even creating an account with yet another web server!
We can have the ease of use of signing in with twitter or facebook without any
centralized servers and very minimal setup from the end user's perspective.

Even better, this authentication system can work offline. In fact, the system
*must* work offline to be fully secure.

# appcache manifesto

By default, web pages can decide to send you whatever javascript and html
payload it wants. This payload is probably what you want, at least at first, but
consider what might happen if those asymmetric keys are used for commerce or for
private communications. Suddenly, the webmaster could easily decide to serve a
different payload, perhaps in a targetted manner, that copies private keys to
third parties or performs malicious operations. Even if the webmaster is an
upstanding person, a government agent could show up at any time with a court
order forcing the webmaster to serve up a different payload for some users.

Imagine if whenever you ran the `ssh` command, your computer fetched the latest
version of the ssh binary from openssh.org and then executed it. This would be
completely unacceptable for server programs, and browser apps that handle
confidental keys and data should be no different!

Luckily, there is another relatively new feature in the browser that can protect
against rogue server updates: the appcache manifest. A page can set a manifest
file with:

```
<html manifest="page.appcache">
```

and then the browser will load a cache policy from `page.appcache`. The
appcache file can be used to make some documents available offline, but can also
be used to prevent the browser from fetching updates to documents. If the
max-age header on the appcache file itself is set far enough in the future, the
appcache file itself can be made permanent so that the server operator can't
update this file either. In the future, the service worker API will provide
enough hooks to do the same thing, but browser support is not widespread yet.

# hyperboot

Upgrading an application should be possible too without going into the bowels of
the browser to clear the appcache. This is where
[hyperboot](http://hyperboot.org) comes in to give us opt-in application
upgrades for end-users. More security-minded users might even want to check with
external auditing systems before upgrading.

# keyboot

With a versioning system in place, we can now start implementing an offline
single sign-on system that exposes the
[web crypto methods](http://msdn.microsoft.com/en-us/library/ie/dn302321(v=vs.85).aspx)
securely without exposing private keys to random websites.

There are another few nifty tricks with the service worker API that can give us
[realtime communication between tabs and
iframes](https://npmjs.org/package/page-bus)
that works completely offline.

To give this new system a try, first open
[https://keyboot.org](https://keyboot.org) in a modern browser and generate a
key.

Next open up
[http://keyboot-example-app.hyperboot.org/](http://keyboot-example-app.hyperboot.org/)
in a new window or tab and paste `https://keyboot.org/` into the text box.
In the https://keyboot.org/ window, approve the request. Now from
`http://keyboot-example-app.hyperboot.org/`, you can sign messages with your
private key!

There is [still plenty to do](https://github.com/substack/keyboot/issues) and
some unanswered questions about different threat models and how best to prevent
replay attacks and domain isolation, but this proof of concept should be good
enough to at least start people thinking about decentralized approaches to
single sign-on and the changing role of servers and webapps as browser APIs
become more capable.

# links

* [http://hyperboot.org](http://hyperboot.org)
* [https://github.com/substack/hyperboot](https://github.com/substack/hyperboot)
* [https://github.com/substack/keyboot](https://github.com/substack/keyboot)
* [https://github.com/substack/keyboot-example-app](https://github.com/substack/keyboot-example-app)
* [http://keyboot-example-app.hyperboot.org/](http://keyboot-example-app.hyperboot.org/)
