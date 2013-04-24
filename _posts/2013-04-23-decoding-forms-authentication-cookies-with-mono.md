---
title: Decoding Forms Authentication Cookies With Mono
layout: post
---

Lately, I've been experimenting with migrating web services from .NET on
Windows to Mono on Ubuntu. While getting the code building and running
in a VM was not terribly difficult, I soon found that authentication was
a roadblock.

Our web services run across several subdomains. We're moving towards
using OAuth for everything, but many services still use Forms
Authentication with a cookie that is shared across all subdomains.
However, when I tried to use my cookie with a service running on Mono, I
was treated as an unauthenticated user.

The first barrier to shared authentication between .NET and Mono is
actually
[well-documented](http://www.mono-project.com/Config_system.web_authentication):
the default cookie name under Mono is `.MONOAUTH`, rather than
`.ASPXAUTH`.
Upon discovering this, I quickly modified my Web.config to explicitly
set the cookie to `.ASPXAUTH`…and nothing happened.

Another early discovery was that Mono base-64 encodes the cookie, while
.NET uses base-16. Again, changing this (in my case, by using a custom
build of Mono) had no effect—I was still treated as an unauthenticated
user.

Eventually, after quite a bit of digging around, I learned that the
binary format for .NET's Forms Authentication cookie is *undocumented*,
so Mono had to implement a reasonable—yet incompatible—alternative
format. Furthermore, if you choose to encrypt the cookie, .NET adds an
extra 28 bytes of padding to the cookie using an unspecified hash
function—my best guess is that this has something to do with avoiding
[hash collision
attacks](http://weblogs.asp.net/scottgu/archive/2011/12/28/asp-net-security-update-shipping-thursday-dec-29th.aspx).

In any case, I spent a lazy Saturday afternoon poking around with the
format and arrived at a method of decoding the cookie, which is included
here for your reading pleasure:

<script src='https://gist.github.com/david-mitchell/5350992.js'>
</script>
<noscript>If you had Javascript enabled, you'd see a <a href='https://gist.github.com/david-mitchell/5350992'>gist</a> here.</noscript>

As you can see, it's very proof-of-concepty, and it assumes specific
encryption and signing algorithms, but it gets the job done and should
be pretty easy to extend and generalize. Just drop this code into your
Global.asax.cs, provide your favorite implementation of base-16 string
to bytes, and you're good to go.

Happy hacking!

