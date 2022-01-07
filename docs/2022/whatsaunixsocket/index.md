# Unix sockets, the basics in Rust


I found myself wondering about unix sockets
while working on [Sozu](https://github.com/sozu-proxy/sozu),
a reverse proxy written in Rust.
A bunch of Sōzu issues led me to
[dig into Sōzu channels](https://github.com/Keksoj/stream_stuff_on_a_sozu_channel),
which themselves make use of
[Metal I/O 's implementation of unix sockets](https://tokio-rs.github.io/mio/doc/mio/net/struct.UnixListener.html).

Here the questions, summed up:

-   what is a unix socket?
-   how can we create one in Rust?
-   how do we use it to stream data?

So here we go.

## What is a unix socket?

It is _not_ a web socket like `127.0.0.1:8080`.

You may have heard that in unix,
[everything is a file](https://www.wikiwand.com/en/Everything_is_a_file).
Unix sockets seem to be a good example of this principle.
They are empty files of sorts, only there to be written onto, and read from.

Sockets seem to be a core feature of unix. In fact, if you type

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

The Rust standard library has a [unix module](https://doc.rust-lang.org/std/os/unix/index.html)
that contains modules to interact with unix processes, unix files, and so on.
Within this unix module, we want to look at the `net` module,
because unix sockets are used to do networking between processes.

The `std::os::unix::net` module contains, among other things:

-   [`UnixListener`](https://doc.rust-lang.org/std/os/unix/net/struct.UnixListener.html)
-   [`UnixStream`](https://doc.rust-lang.org/std/os/unix/net/struct.UnixStream.html)

Both those entities are unsafe wrappers of the `libc` library to perform the very same unix system calls you would write in C.
They both wrap a unix file descriptor, but they are distinct in order to separate higher-level concerns.
`UnixListener` is used to create sockets, `UnixStream` is there to read from and write on them.

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

    rm mysocket

## Waiting for connections, server side

The `UnixListener` struct has an `accept()` method that waits for other processes to connect to the socket.
Once a connections come, `accept()` returns a tuple containing a `UnixStream` and a `SocketAddr`.

As mentioned above, `UnixStream` implements `Read` and `Write`.
We will handle this stream to:

-   read what another process would write on the socket
-   write responses

Complete the code:

```rust
// src/bin/server.rs
use std::os::unix::net::{UnixListener, UnixStream};

fn main() -> anyhow::Result<()> {
    let socket_path = "mysocket";

    let unix_listener =
        UnixListener::bind(socket_path).context("Could not create the unix socket")?;

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

    rm mysocket
    cargo run --bin server

it should hang.
Perfect! The server is waiting for connections!

## Connecting to the socket, client side

The client process wants to connect to an existing socket, read an write from it.

Next to `server.rs`, create the `client.rs` file.
The client will merely consist of a `UnixStream`:

```rust
// src/bin/client.rs
use std::os::unix::net::{UnixListener, UnixStream};

use anyhow::Context;

fn main() -> anyhow::Result<()> {
    let socket_path = "mysocket";

    let mut unix_stream = UnixStream::connect(socket_path).context("Could not create stream")?;

    Ok(())
```

## Writing onto the socket, client side

We need to import the `Read` and `Write` traits, and now we can write onto the stream.
Complete the code:

```rust
// src/bin/client.rs
use std::io::{Read, Write};
use std::os::unix::net::{UnixListener, UnixStream};

use anyhow::Context;

fn main() -> anyhow::Result<()> {
    let socket_path = "mysocket";

    let mut unix_stream = UnixStream::connect(socket_path).context("Could not create stream")?;

    unix_stream
        .write(b"Hello?")       // we write bytes &[u8]
        .context("Failed at writing onto the unix stream")?;

    Ok(())
}
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
fn handle_stream(mut stream: UnixStream) -> anyhow::Result<()> {
    let mut message = String::new();
    stream
        .read_to_string(&mut message)
        .context("Failed at reading the unix stream")?;

    println!("{}", message);
    Ok(())
}
```

### Launch the whole thing!

Make sure you have the server running in a terminal:

    rm mysocket
    cargo run --bin server

And in a separate terminal, run the client:

    cargo run --bin client

If all is well, the hello message should display on the server side.

## The associated repository

This tutorial comes with a [github repository](https://github.com/Keksoj/unix_socket_based_server_client)
with a little [custom-made socket library](https://github.com/Keksoj/unix_socket_based_server_client/blob/main/src/socket.rs)
that does the automatic deletion and some more things like setting permissions.

```

```

