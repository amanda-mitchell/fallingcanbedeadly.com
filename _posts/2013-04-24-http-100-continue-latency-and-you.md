---
title: HTTP 100 Continue, Latency, and You
layout: post
---

### TL;DR

You probably shouldn't send an `Expect: 100-continue` header in your HTTP
requests—especially if you're making requests against a server running
IIS or Nginx. If you're using .NET, you have to explicitly opt out of
sending this header by setting `Expect100Continue` to `false` on the
`ServicePoint` of any `HttpWebRequest` that you create.

### Background

In a recent project, I found myself digging into the details surrounding
the [100 Continue
status](http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html#sec8.2.3)
in HTTP 1.1. The project involves uploading (potentially) large files to
the server, and it initially seemed like explicitly supporting `100
Continue` would be a good way to preserve bandwidth.

### The Problem

First off, there's surprisingly little reliable documentation about this status
code beyond RFC 2616 itself. IIS automatically sends a `100 Continue` for
*any* request containing an `Expect: 100-Continue` header, and it wasn't
 clear from my research whether there is actually a way to disable
this—at the very least, it's not possible from within the context of a
plain old ASP.NET application.

As I investigated further, I discovered that it's actually somewhat
common for servers to handle this without allowing application code to
have a say in what happens. Nginx's reverse proxy module behaves this
way, and projects like Gunicorn even [depend on this
behavior](http://docs.gunicorn.org/en/latest/deploy.html#nginx-configuration)
in order to prevent a [certain class](http://ha.ckers.org/slowloris/) of
DOS attack.

Normally, this would be the end of my investigation: what initially
seemed like a cool feature would be a pain to implement, and after
reading up on it more, it really seemed to be of dubious value. But then
I started reading up on related parts of the .NET framework…

### HttpWebRequest

It seems that the designers of the .NET framework weren't allowed to talk to the
developers of IIS. If they had, the .NET framework designers might have
noticed that sending an `Expect: 100-Continue` header to servers that
*always* respond with `100 Continue` serves *absolutely no purpose*. In
 fact, the current behavior is *worse* than useless because it
introduces unnecessary latency any time that you call out to an external
web service. For an application that makes an occasional web request,
this might not be so bad, but inside of a web service with cross-service
dependencies, or on a mobile device with naturally high latency, this is
a big deal.

Fortunately, [this header can be
disabled](http://haacked.com/archive/2004/05/15/http-web-request-expect-100-continue.aspx): every `HttpWebRequest` has a
`ServicePoint` with an `Expect100Continue` property. If you set this to
`false`, you'll save yourself an unnecessary round trip. Better yet:
write yourself a little utility method that creates an `HttpWebRequest`
and disables the header, then flame anyone who doesn't use it.

Happy Hacking!

