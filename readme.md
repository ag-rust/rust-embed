## Rust Embed [![Build Status](https://travis-ci.org/pyros2097/rust-embed.svg?branch=master)](https://travis-ci.org/pyros2097/rust-embed) [![Build status](https://ci.appveyor.com/api/projects/status/jnr45fn1l1dop2a8/branch/master?svg=true)](https://ci.appveyor.com/project/pyros2097/rust-embed/branch/master) [![crates.io](http://meritbadge.herokuapp.com/rust-embed)](https://crates.io/crates/rust-embed)

Rust Custom Derive Macro which loads files into the rust binary at compile time during release and loads the file from the fs during dev.

You can use this to embed your css, js and images into a single executable which can be deployed to your servers. Also it makes it easy to build a very small docker image for you to deploy.

### Dev

<img src="https://user-images.githubusercontent.com/1687946/40840773-b1ae1ce6-65c5-11e8-80ac-9e9196701ca2.png" width="700" height="100">

### Release

<img src="https://user-images.githubusercontent.com/1687946/40840774-b1dd709a-65c5-11e8-858d-73a88e25f07a.png" width="700" height="184">

## Installation

```toml
[dependencies]
rust-embed="5.5.1"
```

## Documentation

You need to add the custom derive macro RustEmbed to your struct with an attribute `folder` which is the path to your static folder.

The path resolution works as follows:

- In `debug` and when `debug-embed` feature is not enabled, the folder path is resolved relative to where the binary is run from.
- In `release` or when `debug-embed` feature is enabled, the folder path is resolved relative to where `Cargo.toml` is.

```rust
#[derive(RustEmbed)]
#[folder = "examples/public/"]
struct Asset;
```

The macro will generate the following code:

```rust
impl Asset {
  pub fn get(file_path: &str) -> Option<Cow<'static, [u8]>> {
    ...
  }

  pub fn iter() -> impl Iterator<Item = Cow<'static, str>> {
    ...
  }
}
impl RustEmbed for Asset {
  fn get(file_path: &str) -> Option<Cow<'static, [u8]>> {
    ...
  }
  fn iter() -> impl Iterator<Item = Cow<'static, str>> {
    ...
  }
}
```

### `get(file_path: &str)`

Given a relative path from the assets folder returns the bytes if found.

If the feature `debug-embed` is enabled or the binary compiled in release mode the bytes have been embeded in the binary and a `Cow::Borrowed(&'static [u8])` is returned.

Otherwise the bytes are read from the file system on each call and a `Cow::Owned(Vec<u8>)` is returned.

### `iter()`

Iterates the files in this assets folder.

If the feature `debug-embed` is enabled or the binary compiled in release mode a static array to the list of relative paths to the files is returned.

Otherwise the files are listed from the file system on each call.

## Features

### `debug-embed`

Always embed the files in the binary, even in debug mode.

### `interpolate-folder-path`

Allow environment variables to be used in the `folder` path. Example:

```rust
#[derive(RustEmbed)]
#[folder = "$CARGO_MANIFEST_DIR/foo"]
struct Asset;
```

This will pull the `foo` directory relative to your `Cargo.toml` file.

### `compression`

Compress each file when embedding into the binary. Compression is done via [`include-flate`].

## Usage

```rust
use rust_embed::RustEmbed;

#[derive(RustEmbed)]
#[folder = "examples/public/"]
struct Asset;

fn main() {
  let index_html = Asset::get("index.html").unwrap();
  println!("{:?}", std::str::from_utf8(index_html.as_ref()));

  for file in Asset::iter() {
      println!("{}", file.as_ref());
  }
}
```

## Examples

To run the example in dev mode where it reads from the fs,

`cargo run --example basic`

To run the example in release mode where it reads from binary,

`cargo run --example basic --release`

Note: To run the `actix-web` example:

`cargo run --example actix --features actix`

Note: To run the `rocket` example, add the `nightly` feature flag and run on a nightly build:

`cargo +nightly run --example rocket --features nightly`

Note: To run the `warp` example:

`cargo run --example warp --features warp-ex`

## Testing

debug: `cargo test --test lib`

release: `cargo test --test lib --release`

Go Rusketeers!
The power is yours!

[`include-flate`]: https://crates.io/crates/include-flate
