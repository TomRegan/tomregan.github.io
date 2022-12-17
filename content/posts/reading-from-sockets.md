---
title: Reading from Sockets
date: 2012-09-22
---

This is a story about how I learned to read streams from sockets, or
to put it another way:

{% highlight diff %}
-  strncpy(msg_buf, "HTTP/1.0 404 Not Found\r\n", HTTP_RESPONSE_LEN);
+  strncpy(msg_buf, "HTTP/1.1 404 Not Found\r\n", HTTP_RESPONSE_LEN);
{% endhighlight %}

How I learned to stop worrying and begin to implement `HTTP/1.1`.

That's [pushy](https://github.com/TomRegan/pushy). All it knows is [it doesn't know](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.4).

## HTTP/1.0

For most of the practical work I've done with sockets until now
I've used datagrams, so I decided that all socket
reads, including streaming connections, work in the same way as
HTTP/1.0, which is to open a new
connection for every request. The code I used to implement
my reads was something like this:

{% highlight c %}
int                     fd, new_fd;
char                    buf [REQUEST_BUFFER_LEN +1]; /* 1023 + 1 */
socklen_t               size = sizeof(struct sockaddr_in);

/* a bunch of socket initialization */

for (;;) {
    new_fd = accept(fd, (struct sockaddr *)&peer_addr, (socklen_t *) &size);
    /* fork  */
    recv(new_fd, buf, REQUEST_HEAD_LEN, 0);
    /* process request and send response */
    close(new_fd);
}
{% endhighlight %}

It turns out this is almost entirely wrong.

## HTTP/1.1

When I sent an HTTP/1.1 header I found that the server would regularly
block on `accept` even though the client was waiting on a response.

### close
Calling `close` on the server is more involved that is immediately
apparent, as *pushes glasses up nose* [RFC 2616](https://tools.ietf.org/html/rfc2616#section-8.1.2)
clearly states...

> Persistent connections provide a mechanism by which a client and a server can signal the close of a TCP connection. This signaling takes place using the Connection header field section 14.10.  Once a close has been signaled, the client MUST NOT send any more requests on that connection.

it continues in ([14.10](https://tools.ietf.org/html/rfc2616#section-14.10))...

> HTTP/1.1 defines the "close" connection option for the sender to
> signal that the connection will be closed after completion of the
> response. For example,
>     Connection: close

So the correct thing to do before closing a connection is to send
`Connection: close` in the header, and not quietly close after each
response like I had been doing.

The second bug I found was a sound lesson in RTFM. Here are the
salient points:

> "[Accpet] creates a new socket.... The original socket remains open."

I did a bit of mental arithmatic, and I counted two sockets in total.
I went back to the code and reviewed it pretty extensively, and found I was
only ever closing the new socket desctiptor returned by `accept`.

The bug was obvious when I checked `netstat` and found dozens of
sockets held in `CLOSE_WAIT`. This is the result of the client closing the
connection, but the server failing to respond.

This demonstrates the solution as it's implemented in pushy, where
`sockfd` is the listening socket, and `peerfd` is the connected host.

{% highlight c %}
if ((pid = fork()) == 0) {
    close(sockfd);
    log_conn(FINE, &peer_addr, "connected\n");
    accept_request(peerfd, &peer_addr);
    log_conn(FINE, &peer_addr, "done serving request\n");
    exit(0);
}
log_ln(DEBUG, "forked process %i\n", pid);
close(peerfd);
{% endhighlight %}

There are lots of good resources. I've been referring to [RFC 2616](https://tools.ietf.org/html/rfc2616#section-8.1.2)
as I go. The man pages for sockets are easy to read, and nicely
augmented by a couple of papers on BSD4.4 IPC,

* An Introductory 4.4BSD Interprocess Communication Tutorial
* An Advanced 4.4BSD Interprocess Communication Tutorial
