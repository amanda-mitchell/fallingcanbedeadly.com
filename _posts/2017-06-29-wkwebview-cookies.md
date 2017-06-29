---
title: Send Cookies to a WKWebView With This One Weird Trick
layout: post
---

Back in iOS 8, Apple introduced a new control for hosting web content: `WKWebView`.
In most respects, it's far superior to the old `UIWebView`:
it's faster, has fewer rendering issues, and provides better Javascript support.

However, one area in which `WKWebView` is demonstrably worse is in its handling of cookies.
iOS provides a service called `NSHTTPCookieStorage` that is used by most HTTP-related functionality, including `UIWebView`—but not `WKWebView`.

The reason for this is documented in a [long-lived webkit bug](https://bugs.webkit.org/show_bug.cgi?id=140191).
In `WKWebView`, most of the underlying functionality actually lives in a process outside of your application.
Among other benefits, this policy allows Apple to use a JavaScript engine that produces executable code at runtime without giving all processes this privilege.
Unfortunately, this complicates interactions with things that *do* live in your application—like cookie stores.
While this issue *has* been addressed on master, it's not yet in a shipping version of iOS. (it looks like iOS 11 may provide a way to interact with `WKWebView`'s cookie store)

For apps that host web applications that depend on cookies for authentication, this poses a major problem.
Although web requests originating in the application can access the shared cookie store, these cookies are never propagated to the web view, meaning that subsequent AJAX requests or page navigations will be unauthenticated.

The most popular solution on [the bathroom wall](https://stackoverflow.com/questions/26573137/can-i-set-the-cookies-to-be-used-by-a-wkwebview) appears to be injecting extra Javascript into each page to programmatically set the cookie.
While this works, it requires dynamically generating Javascript, which is kind of gross.

If you have control of the server hosting your web content, there's another way to slip your cookies into a `WKWebView`:
just get the server to send back a `Set-Cookie` header in its response to your initial navigation request.
In the case of an app that I was recently working on, we added a small bit of code on the server looking for an `X-Echo-Cookie` HTTP header.
For any request containing this header, it would include a `Set-Cookie` header whose content was whatever was in the request's `Cookie` header.

By including this header on our initial navigation request, we were able to get the cookie to "jump" from the application's cookie store to `WKWebView`'s.
While this solution doesn't cover complex scenarios, like the cookie being updated or invalidated while the user is navigating, it worked just fine for our scenario.
