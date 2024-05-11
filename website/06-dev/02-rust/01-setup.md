---
layout: page
title: Инсталляция rust в linux
description: Инсталляция rust в linux
keywords: программирование, языки, rust, инсталляция rust в linux
permalink: /dev/rust/setup/
---

<br/>

# Инсталляция rust в linux

<br/>

**Делаю:**  
2024.05.10

<br/>

```
$ cd ~/tmp/
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

$ source $HOME/.cargo/env

$ rustup update
```

<br/>

```
$ rustc --version
rustc 1.78.0 (9b00956e5 2024-04-29)

$ cargo --version
cargo 1.78.0 (54d8815d0 2024-03-26)

$ rustup --version
rustup 1.27.1 (54dd3d00f 2024-04-24)
```

<br/>

```
$ rustup show
Default host: x86_64-unknown-linux-gnu
rustup home:  /home/marley/.rustup

stable-x86_64-unknown-linux-gnu (default)
rustc 1.78.0 (9b00956e5 2024-04-29)
```

<br/>

```
// Использовать последнюю версию rust
// $ rustup override set nightly

// Использовать стабильную версию rust
$ rustup override set stable
```

<br/>

### Дополнительные пакеты

<br/>

```
// Linting
$ rustup component add clippy

// Formatting
$ rustup component add rustfmt
```

<br/>

```
// Monitors your source code to trigger commands every time a file changes
$ cargo install cargo-watch

// Code Coverage
$ sudo apt install -y pkg-config libssl-dev
$ cargo install cargo-tarpaulin

// Security Vulnerabilities
$ cargo install cargo-audit
```

<br/>

### rust-analyzer

https://github.com/rust-analyzer/rust-analyzer

<br/>

```
$ cd ~/apps/
$ git clone https://github.com/rust-analyzer/rust-analyzer
$ cd rust-analyzer/

$ rustup update
$ cargo xtask install --server
$ rust-analyzer --version
```

<br/>

### VSCode extensions

<br/>

```
// rust-analyzer
ext install rust-lang.rust-analyzer
```

- Rust (Deprecated заменен на rust-analyzer)
- CodeLLDB
- crates
- Even Better TOML
- rust-analyzer

<br/>

```
^P
> Settings JSON
```

<br/>

```
"rust-analyzer.check.command": "clippy",
```
