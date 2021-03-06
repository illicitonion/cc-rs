# cc-rs

A library to compile C/C++/assembly into a Rust library/application.

[![Build Status](https://travis-ci.org/alexcrichton/cc-rs.svg?branch=master)](https://travis-ci.org/alexcrichton/cc-rs)
[![Build status](https://ci.appveyor.com/api/projects/status/onu270iw98h81nwv?svg=true)](https://ci.appveyor.com/project/alexcrichton/cc-rs)

[Documentation](https://docs.rs/cc)

A simple library meant to be used as a build dependency with Cargo packages in
order to build a set of C/C++ files into a static archive. This crate calls out
to the most relevant compiler for a platform, for example using `cl` on MSVC.

> **Note**: this crate was recently renamed from the `gcc` crate, so if you're
> looking for the `gcc` crate you're in the right spot!

## Using cc-rs

First, you'll want to both add a build script for your crate (`build.rs`) and
also add this crate to your `Cargo.toml` via:

```toml
[build-dependencies]
cc = "1.0"
```

Next up, you'll want to write a build script like so:

```rust,no_run
// build.rs

extern crate cc;

fn main() {
    cc::Build::new()
        .file("foo.c")
        .file("bar.c")
        .compile("foo");
}
```

And that's it! Running `cargo build` should take care of the rest and your Rust
application will now have the C files `foo.c` and `bar.c` compiled into a file
named libfoo.a. You can call the functions in Rust by declaring functions in
your Rust code like so:

```
extern {
    fn foo_function();
    fn bar_function();
}

pub fn call() {
    unsafe {
        foo_function();
        bar_function();
    }
}

fn main() {
    // ...
}
```

## External configuration via environment variables

To control the programs and flags used for building, the builder can set a
number of different environment variables.

* `CFLAGS` - a series of space separated flags passed to compilers. Note that
             individual flags cannot currently contain spaces, so doing
             something like: "-L=foo\ bar" is not possible.
* `CC` - the actual C compiler used. Note that this is used as an exact
         executable name, so (for example) no extra flags can be passed inside
         this variable, and the builder must ensure that there aren't any
         trailing spaces. This compiler must understand the `-c` flag. For
         certain `TARGET`s, it also is assumed to know about other flags (most
         common is `-fPIC`).
* `AR` - the `ar` (archiver) executable to use to build the static library.

Each of these variables can also be supplied with certain prefixes and suffixes,
in the following prioritized order:

1. `<var>_<target>` - for example, `CC_x86_64-unknown-linux-gnu`
2. `<var>_<target_with_underscores>` - for example, `CC_x86_64_unknown_linux_gnu`
3. `<build-kind>_<var>` - for example, `HOST_CC` or `TARGET_CFLAGS`
4. `<var>` - a plain `CC`, `AR` as above.

If none of these variables exist, cc-rs uses built-in defaults

In addition to the the above optional environment variables, `cc-rs` has some
functions with hard requirements on some variables supplied by [cargo's
build-script driver][cargo] that it has the `TARGET`, `OUT_DIR`, `OPT_LEVEL`,
and `HOST` variables.

[cargo]: http://doc.crates.io/build-script.html#inputs-to-the-build-script

## Optional features

Currently cc-rs supports parallel compilation (think `make -jN`) but this
feature is turned off by default. To enable cc-rs to compile C/C++ in parallel,
you can change your dependency to:

```toml
[build-dependencies]
cc = { version = "1.0", features = ["parallel"] }
```

By default cc-rs will limit parallelism to `$NUM_JOBS`, or if not present it
will limit it to the number of cpus on the machine. If you are using cargo,
use `-jN` option of `build`, `test` and `run` commands as `$NUM_JOBS`
is supplied by cargo.

## Compile-time Requirements

To work properly this crate needs access to a C compiler when the build script
is being run. This crate does not ship a C compiler with it. The compiler
required varies per platform, but there are three broad categories:

* Unix platforms require `cc` to be the C compiler. This can be found by
  installing cc/clang on Linux distributions and Xcode on OSX, for example.
* Windows platforms targeting MSVC (e.g. your target triple ends in `-msvc`)
  require `cl.exe` to be available and in `PATH`. This is typically found in
  standard Visual Studio installations and the `PATH` can be set up by running
  the appropriate developer tools shell.
* Windows platforms targeting MinGW (e.g. your target triple ends in `-gnu`)
  require `cc` to be available in `PATH`. We recommend the
  [MinGW-w64](http://mingw-w64.org) distribution, which is using the
  [Win-builds](http://win-builds.org) installation system.
  You may also acquire it via
  [MSYS2](http://msys2.github.io), as explained [here][msys2-help].  Make sure
  to install the appropriate architecture corresponding to your installation of
  rustc. GCC from older [MinGW](http://www.mingw.org) project is compatible
  only with 32-bit rust compiler.

[msys2-help]: http://github.com/rust-lang/rust#building-on-windows

## C++ support

`cc-rs` supports C++ libraries compilation by using the `cpp` method on
`Build`:

```rust,no_run
extern crate cc;

fn main() {
    cc::Build::new()
        .cpp(true) // Switch to C++ library compilation.
        .file("foo.cpp")
        .compile("libfoo.a");
}
```

When using C++ library compilation switch, the `CXX` and `CXXFLAGS` env
variables are used instead of `CC` and `CFLAGS` and the C++ standard library is
linked to the crate target.

## License

`cc-rs` is primarily distributed under the terms of both the MIT license and
the Apache License (Version 2.0), with portions covered by various BSD-like
licenses.

See LICENSE-APACHE, and LICENSE-MIT for details.
