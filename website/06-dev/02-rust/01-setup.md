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
2024.07.20

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
rustc 1.79.0 (129f3b996 2024-06-10)

$ cargo --version
cargo 1.79.0 (ffa9cf99a 2024-06-03)

$ rustup --version
rustup 1.27.1 (54dd3d00f 2024-04-24)
```

<br/>

```
$ rustup show
Default host: x86_64-unknown-linux-gnu
rustup home:  /home/marley/.rustup

stable-x86_64-unknown-linux-gnu (default)
rustc 1.79.0 (129f3b996 2024-06-10)
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
```

<br/>

```
$ rust-analyzer --version
rust-analyzer 0.0.0 (b333f85a9 2024-07-19)
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
