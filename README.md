# include_url

[![Crates.io](https://img.shields.io/crates/v/include_url_macro)](https://crates.io/crates/include_url_macro)
[![Documentation](https://docs.rs/include_url_macro/badge.svg)](https://docs.rs/include_url_macro)

A Rust procedural macro that fetches URL content at compile time and includes it as a static string. It also provides JSON parsing capabilities through the `include_json_url` macro.

## Usage

Add this to your `Cargo.toml`:
```toml
[dependencies]
include_url_macro = "0.1.2"
serde = { version = "1.0", features = ["derive"] }  # Required for JSON parsing
serde_json = "1.0"                                  # Required for JSON parsing
```

## Features

This crate supports multiple TLS backend options. Choose the one that best fits your needs:

### Default (native-tls)

Uses the system's native TLS implementation (OpenSSL on Linux, Secure Transport on macOS, SChannel on Windows):
```toml
[dependencies]
include_url_macro = "0.1.2"
```

### Vendored OpenSSL

Compiles and statically links OpenSSL, useful for CI/CD environments or when system OpenSSL is unavailable:
```toml
[dependencies]
include_url_macro = { version = "0.1.2", default-features = false, features = ["native-tls-vendored"] }
```

### Pure Rust TLS (rustls)

Uses rustls, a modern TLS library written in Rust with no C dependencies:
```toml
[dependencies]
include_url_macro = { version = "0.1.2", default-features = false, features = ["rustls"] }
```

**Feature comparison:**

| Feature | Default (`native-tls`) | `native-tls-vendored` | `rustls` |
|---------|------------------------|----------------------|----------|
| System certificates | ✅ Yes | ✅ Yes | ⚠️ Configurable |
| Cross-compilation | ⚠️ Requires system libs | ✅ Easy | ✅ Easy |
| Compile time | ✅ Fast | ⚠️ Slower | ✅ Fast |
| Binary size | ✅ Small | ⚠️ Larger | ✅ Small |
| Dependencies | C (OpenSSL) | C (vendored OpenSSL) | Pure Rust |

### Basic URL Content

Use `include_url` to fetch and include raw content:
```rust
use include_url_macro::include_url;

fn main() {
    // Content will be fetched at compile time
    let readme = include_url!("https://raw.githubusercontent.com/rust-lang/rust/master/README.md");
    println!("{}", readme);
}
```

### JSON Content

Use `include_json_url` to fetch and parse JSON content:
```rust
use include_url_macro::include_json_url;
use serde::Deserialize;

// Parse into serde_json::Value
let json = include_json_url!("https://api.example.com/data.json");
println!("API Version: {}", json["version"]);

// Parse into a specific type
#[derive(Deserialize)]
struct User {
    id: u64,
    name: String,
    email: String,
}

let user = include_json_url!("https://api.example.com/user.json", User);
println!("User name: {}", user.name);
```

## Capabilities

- Fetches URL content at compile time
- Supports HTTP and HTTPS URLs
- Validates URLs before fetching
- Provides meaningful compile-time errors
- Similar usage to the built-in `include_str!` macro
- JSON parsing capabilities:
  - Parse into generic `serde_json::Value`
  - Parse into specific Rust types that implement `serde::Deserialize`
  - Compile-time JSON validation (the type does not get validated at compile time)
- Multiple TLS backend options for flexibility

## Safety

This macro only allows HTTP and HTTPS URLs. It performs URL validation before making any requests.
Note that using this macro will make your build process dependent on network connectivity and the availability of the URLs you're including.

## Error Handling

Both macros provide compile-time errors for:

- Invalid URLs
- Network failures
- Invalid content
- JSON parsing errors (for `include_json_url`)
- Type mismatches when parsing JSON into specific types

Example error messages:
```
error: Invalid URL: relative URL without a base
  --> src/main.rs:4:20
   |
4  |     let data = include_url!("not-a-url");
   |                    ^^^^^^^^^

error: Failed to parse JSON into the specified type
  --> src/main.rs:12:24
   |
12 |     let user = include_json_url!("https://api.example.com/data.json", User);
   |                        ^^^^^^^^^^^^^
```

## License

This project is licensed under the MIT License.