# Couchbase Rust SDK

[![LICENSE](https://img.shields.io/badge/license-Apache%202.0-blue.svg)](LICENSE)
[![Crates.io Version](https://img.shields.io/crates/v/couchbase.svg)](https://crates.io/crates/couchbase)

This is the repository for the official, community supported Couchbase Rust SDK. It is currently a work in progress and built on top of [libcouchbase](https://github.com/couchbase/libcouchbase/).

## Requirements

Make sure to have all [libcouchbase](https://docs.couchbase.com/c-sdk/current/start-using-sdk.html) requirements satisfied to build it properly. Also [bindgen](https://rust-lang.github.io/rust-bindgen/requirements.html) requirements need to be in place. You need a rust version newer or equal to `1.77.2` because this SDK makes heavy use of `async/await`.

## Installation

```toml
[dependencies]
couchbase = "1.0.0-alpha.5"
```

## Dependencies
By default the library depends on:

* _openssl_ for SSL transport.
```shell
brew update
brew install openssl
```
* Set the value of
  `OPENSSL_ROOT_DIR` to the location of the installation path
* Set the value of `PKG_CONFIG_PATH` 
    For example: ```export PKG_CONFIG_PATH="/opt/homebrew/opt/openssl@3/lib/pkgconfig"```
* Download the rustup installation program and use it to install Rust by running the following command:
```shell
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
* Follow the prompts displayed to proceed with a default installation.
* Update your current shell to include Cargo by running the following command:
```shell
source ~/.cargo/env
```

## Building

From the root of the repo:
```shell
cargo build
```

## Testing

### To run only Unit Tests
From the root of the repo:
```shell
cargo test --lib
```
### To run only Integration Tests
From the root of the repo:
```shell
cargo test --test '*'
```

## Usage

The `examples` folder has a bunch more, but here is a basic getting started doing a kv op:

```rust
pub fn main() {
    // Connect to the cluster with a connection string and credentials
    let cluster = Cluster::connect("couchbase://127.0.0.1", "Administrator", "password");
    // Open a bucket
    let bucket = cluster.bucket("travel-sample");
    // Use the default collection (needs to be used for all server 6.5 and earlier)
    let collection = bucket.default_collection();

    // Fetch a document
    match block_on(collection.get("airline_10", GetOptions::default())) {
        Ok(r) => println!("get result: {:?}", r),
        Err(e) => println!("get failed! {}", e),
    };

    // Upsert a document as JSON
    let mut content = HashMap::new();
    content.insert("Hello", "Rust!");

    match block_on(collection.upsert("foo", content, UpsertOptions::default())) {
        Ok(r) => println!("upsert result: {:?}", r),
        Err(e) => println!("upsert failed! {}", e),
    };
}
```

## Examples
More examples can be found in the `examples` folder. Please open a ticket if something is not present or does not showcase what you need.

## Unsafe Code
This code contains **unsafe {}** code blocks. Breathe slowly and calm down, it's going to be okay. The reason why we use unsafe code is so that we can call into `libcouchbase` which is a C library. The only unsafe code is found in the lcb part of the IO module. So if you experience a segfault, it will likely come from there. We are trying to even keep unsafe in there minimal, but by the nature of it, it is all over the place. We are also working on a pure Rust SDK with no unsafe code (hopefully), but until this ships and is mature we have to live with it.