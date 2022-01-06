---
title: "Unix sockets, the basics in Rust"
date: 2022-01-06T18:51:20+01:00
draft: true
---

I found myself wondering about unix sockets
while working on [Sozu](https://github.com/sozu-proxy/sozu),
a reverse proxy written in Rust.
A bunch of Sōzu issues led me to
[dig into Sōzu channels](https://github.com/Keksoj/stream_stuff_on_a_sozu_channel),
which themselves make use of
[Metal I/O 's implementation of unix sockets](https://tokio-rs.github.io/mio/doc/mio/net/struct.UnixListener.html).

Here the questions, summed up:

-   what is a unix socket?
-   what is a unix listener?
-   what is a unix stream?

So here we go.

## What is a unix socket?

As myself, you may have heard that in unix,
[everything is a file](https://www.wikiwand.com/en/Everything_is_a_file).
Unix sockets seem to be a good example of this principle.
Empty files of sorts, only there to be written onto, and read from.
In fact, if you type

    man unix

in your terminal, you should land on an ancient man page:

```
UNIX(7)                 Linux Programmer's Manual                UNIX(7)

NAME

       unix - sockets for local interprocess communication
```


