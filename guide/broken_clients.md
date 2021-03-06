Dealing with broken clients
===========================

There exists a very large number of implementations for the
HTTP protocol. Most widely used clients, like browsers,
follow the standard quite well, but others may not. In
particular custom enterprise clients tend to be very badly
written.

Cowboy tries to follow the standard as much as possible,
but is not trying to handle every possible special cases.
Instead Cowboy focuses on the cases reported in the wild,
on the public Web.

That means clients that ignore the HTTP standard completely
may fail to understand Cowboy's responses. There are of
course workarounds. This chapter aims to cover them.

Lowercase headers
-----------------

Cowboy converts all headers it receives to lowercase, and
similarly sends back headers all in lowercase. Some broken
HTTP clients have issues with that.

A simple way to solve this is to create an `onresponse` hook
that will format the header names with the expected case.

``` erlang
capitalize_hook(Status, Headers, Body, Req) ->
    Headers2 = [{cowboy_bstr:capitalize_token(N), V}
        || {N, V} <- Headers],
    {ok, Req2} = cowboy_req:reply(Status, Headers2, Body, Req),
    Req2.
```

Note that SPDY clients do not have that particular issue
because the specification explicitly says all headers are
lowercase, unlike HTTP which allows any case but treats
them as case insensitive.

Camel-case headers
------------------

Sometimes it is desirable to keep the actual case used by
clients, for example when acting as a proxy between two broken
implementations. There is no easy solution for this other than
forking the project and editing the `cowboy_protocol` file
directly.

Chunked transfer-encoding
-------------------------

Sometimes an HTTP client advertises itself as HTTP/1.1 but
does not support chunked transfer-encoding. This is invalid
behavior, as HTTP/1.1 clients are required to support it.

A simple workaround exists in these cases. By changing the
Req object response state to `waiting_stream`, Cowboy will
understand that it must use the identity transfer-encoding
when replying, just like if it was an HTTP/1.0 client.

``` erlang
Req2 = cowboy_req:set(resp_state, waiting_stream).
```
