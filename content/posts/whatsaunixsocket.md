---
title: "Unix sockets, the basics in Rust"
date: 2022-01-06T18:51:20+01:00
draft: false
---

I found myself wondering about unix sockets
while working on [Sōzu](https://github.com/sozu-proxy/sozu),
a reverse proxy written in Rust.
A bunch of Sōzu issues led me to
[dig into Sōzu channels](https://github.com/Keksoj/stream_stuff_on_a_sozu_channel),
which themselves make use of
[Metal I/O 's implementation of unix sockets](https://tokio-rs.github.io/mio/doc/mio/net/struct.UnixListener.html).

Here are the questions, summed up:

-   what are unix sockets?
-   how can we create them in Rust?
-   how do we use them to stream data?

So here we go.

## What is a unix socket?

It is _not_ a web socket like `127.0.0.1:8080`.

You may have heard that in unix,
[everything is a file](https://www.youtube.com/watch?v=dDwXnB6XeiA).
Unix sockets seem to be a good example of this principle.
They are empty files of sorts, only there to be written to, and read from.

Sockets are a core feature of unix. In fact, if you type

    man unix

in your terminal, you should land on an ancient man page:

```
UNIX(7)                 Linux Programmer's Manual                UNIX(7)

NAME

       unix - sockets for local interprocess communication
```

that explains how sockets are declared in C in the kernel,
how they are created with the `AF_UNIX` system call,
and many more thing that go far beyond my limited understanding.

Creating a socket is not as easy as creating just any file, using, say, `touch`.
They are tools available in the command line, but most of the time,
sockets are created and used by processes, not by users.
Looking up how to create one will land you on a tutorial in C, or in python.
So let's see how to do it in Rust.

# Make a socket server in Rust

The Rust standard library has a [std::os::unix module](https://doc.rust-lang.org/std/os/unix/index.html)
to interact with unix processes, unix files, and so on.
Within it, we want to look at the `net` module,
named that way because unix sockets are used to do networking between processes.

The `std::os::unix::net` module contains, among other things:

-   [`UnixListener`](https://doc.rust-lang.org/std/os/unix/net/struct.UnixListener.html)
-   [`UnixStream`](https://doc.rust-lang.org/std/os/unix/net/struct.UnixStream.html)

Both those entities are unsafe wrappers of the `libc` library to perform the very same unix system calls you would write in C.
They both wrap a unix file descriptor, but they are distinct in order to separate higher-level concerns.

-   `UnixListener` is used to create sockets, (`libc::bind()` and `libc::listen()`)
-   `UnixStream` is there to connect to a socket (`libc::connect()`), to read from it and write on it.

Let's use those.
[Install Rust and Cargo](https://www.rust-lang.org/tools/install),
[Learn the basics of Rust](https://doc.rust-lang.org/book/),
and then do:

    cargo new unix_sockets

Add this to `Cargo.toml` (makes error propagation easier):

```toml
# Cargo.toml
anyhow = "^1.0.42"
```

## Create a socket, server side

In the `src` directory, create a `bin` directory, in which you will create a `server.rs` file.

```rust
// src/bin/server.rs
use std::os::unix::net::{UnixListener, UnixStream};

use anyhow::Context;

fn main() -> anyhow::Result<()> {
    let socket_path = "mysocket";

    let unix_listener =
        UnixListener::bind(socket_path).context("Could not create the unix socket")?;

    Ok(())
}
```

Then do

    cargo run --bin server

Which should run smoothly, and then do `ls -l` in your directory,
you should have a line like this:

    srwxr-xr-x 1 emmanuel users    0 Jan  7 13:08 mysocket

The `s` stands for _socket_. Congratulations!

Do one more `cargo run --bin server` and you have a neat, self-explanatory OS error:

```
Error: Could not create the unix socket

Caused by:
    Address already in use (os error 98)
```

I guess we'll have to destroy it and recreate it each time.

```rust
// src/bin/server.rs
use std::os::unix::net::{UnixListener, UnixStream};

use anyhow::Context;

fn main() -> anyhow::Result<()> {
    let socket_path = "mysocket";

    // copy-paste this and don't think about it anymore
    // it will be hidden from there on
    if std::fs::metadata(socket_path).is_ok() {
        println!("A socket is already present. Deleting...");
        std::fs::remove_file(socket_path).with_context(|| {
            format!("could not delete previous socket at {:?}", socket_path)
        })?;
    }

    let unix_listener =
        UnixListener::bind(socket_path).context("Could not create the unix socket")?;

    Ok(())
}
```

## Waiting for connections, server side

The `UnixListener` struct has an `accept()` method that waits for other processes to connect to the socket.
Once a connections comes, `accept()` returns a tuple containing a `UnixStream` and a `SocketAddr`.

As mentioned above, `UnixStream` implements `Read` and `Write`.
We will handle this stream to:

-   read what another process will send through the socket
-   write responses on the socket

Add the loop and the `handle_stream` function to the server code:

```rust
// src/bin/server.rs
use std::os::unix::net::{UnixListener, UnixStream};

fn main() -> anyhow::Result<()> {
    let socket_path = "mysocket";

    let unix_listener =
        UnixListener::bind(socket_path).context("Could not create the unix socket")?;

    // put the server logic in a loop to accept several connections
    loop {
        let (mut unix_stream, socket_address) = unix_listener
            .accept()
            .context("Failed at accepting a connection on the unix listener")?;
        handle_stream(unix_stream)?;
    }
    Ok(())
}

fn handle_stream(mut stream: UnixStream) -> anyhow::Result<()> {
    // to be filled
    Ok(())
}
```

Remove the existing socket and run the code:

    cargo run --bin server

it should hang.
Perfect! The server is waiting for connections!

## Connecting to the socket, client side

The client process wants to connect to an existing socket, read and write from it.

Next to `server.rs`, create the `client.rs` file.
The client will merely consist of a `UnixStream`:

```rust
// src/bin/client.rs
use std::os::unix::net::{UnixListener, UnixStream};

use anyhow::Context;

fn main() -> anyhow::Result<()> {
    let socket_path = "mysocket";

    let mut unix_stream =
        UnixStream::connect(socket_path).context("Could not create stream")?;

    Ok(())
```

## Writing on the socket, client side

We need to import the `Read` and `Write` traits.

```rust
// src/bin/client.rs
use std::io::{Read, Write};
```

And now we can write onto the stream.
Below the `unix_stream` declaration, add the write logic:

```rust
unix_stream
    .write(b"Hello?")       // we write bytes, &[u8]
    .context("Failed at writing onto the unix stream")?;
```

## Reading from the socket, server side

Be sure to import `Read` and `Write` in `server.rs`:

```rust
// src/bin/server.rs
use std::io::{Read, Write};
```

Now let's fill the `handle_stream` function with ordinary read logic:

```rust
// src/bin/server.rs
fn handle_stream(mut unix_stream: UnixStream) -> anyhow::Result<()> {
    let mut message = String::new();
    unix_stream
        .read_to_string(&mut message)
        .context("Failed at reading the unix stream")?;

    println!("{}", message);
    Ok(())
}
```

### Launch the whole thing!

Make sure you have the server running in a terminal:

    cargo run --bin server

And in a separate terminal, run the client:

    cargo run --bin client

If all is well, the hello message should display on the server side.

## Respond to a message, server side

Let's answer something every time the server receives anything.

```rust
// src/bin/server.rs
fn handle_stream(mut unix_stream: UnixStream) -> anyhow::Result<()> {
    let mut message = String::new();
    unix_stream
        .read_to_string(&mut message)
        .context("Failed at reading the unix stream")?;

    println!("We received this message: {}\nReplying...", message);

    unix_stream
        .write(b"I hear you!")
        .context("Failed at writing onto the unix stream")?;

    Ok(())
}
```

## Listen to responses, client side

Introducing the same reading logic we used on the server **will not work**. Why?
After writing on a stream, we need to shut down the writing, if we want to read from it.

Let's segregate the write and read logic into distinct functions.
Oh, and we pass mutable references (`&mut`) of the unix stream to the function, because… Rust.
Don't worry about it.

```rust
// src/bin/client.rs
use std::io::{Read, Write};
use std::os::unix::net::{UnixListener, UnixStream};

use anyhow::Context;

fn main() -> anyhow::Result<()> {
    let socket_path = "mysocket";

    let mut unix_stream =
        UnixStream::connect(socket_path).context("Could not create stream")?;

    write_request_and_shutdown(&mut unix_stream)?;
    read_from_stream(&mut unix_stream)?;
    Ok(())
}
```

The `shutdown()` method takes a `Shutdown` enum we would otherwise use on TCP streams.
Write below the main function:

```rust
fn write_request_and_shutdown(unix_stream: &mut UnixStream) -> anyhow::Result<()> {
    unix_stream
        .write(b"Hello?")
        .context("Failed at writing onto the unix stream")?;

    println!("We sent a request");
    println!("Shutting down writing on the stream, waiting for response...");

    unix_stream
        .shutdown(std::net::Shutdown::Write)
        .context("Could not shutdown writing on the stream")?;

    Ok(())
}

```

The stream is now clean to be read from.

```rust
fn read_from_stream(unix_stream: &mut UnixStream) -> anyhow::Result<()> {
    let mut response = String::new();
    unix_stream
        .read_to_string(&mut response)
        .context("Failed at reading the unix stream")?;

    println!("We received this response: {}", response);
    Ok(())
}
```

### Launch the whole thing, again!

Have the server running in a terminal:

    cargo run --bin server

And in a separate terminal, run the client:

    cargo run --bin client

If all is well,

-   the hello message should display on the server side
-   the "I hear you" response should display on the client side

You can run the client as many times as you want, since the server runs in a loop.

## Browse the code

This tutorial comes with a [github repository](https://github.com/Keksoj/unix_sockets_basics)
that contains the above code.

Feel free to write an issue for any comment, criticism, or complaint you may have.
Fork and do pull requests as you please.

This blog post is a sum-up of what I learned trying to understand unix sockets while working on Sōzu.
A more elaborate version of the code is available
[in this other repo](https://github.com/Keksoj/unix_socket_based_server_client),
with additional features:

-   a `UnixListener`-wrapping library with a glorious `SocketBuilder` helper (permissions! blocking/nonblocking!)
-   a `Message` module with serializable `Request` and `Response` structs. The Response has a status that is either `Ok`, `Error` or `Processing`
-   a client loop that continues reading the stream as long as responses come with a `Processing` status, to stops only at `Ok` or `Error`

All this happened thanks to my employer, [Clever Cloud](https://clever-cloud.com/),
who allows me to learn my job in the best possible conditions. Much gratitude.
