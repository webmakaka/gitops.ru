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
2024.09.28

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
rustc 1.81.0 (eeb90cda1 2024-09-04)

$ cargo --version
cargo 1.81.0 (2dbb1af80 2024-08-20)

$ rustup --version
rustup 1.27.1 (54dd3d00f 2024-04-24)
```

<br/>

```
$ rustup show
Default host: x86_64-unknown-linux-gnu
rustup home:  /home/marley/.rustup

stable-x86_64-unknown-linux-gnu (default)
rustc 1.81.0 (eeb90cda1 2024-09-04)
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
rust-analyzer 0.0.0 (546339a7b 2024-09-27)
```

<br/>

### VSCode extensions

<br/>

```
Ctrl+P

// rust-analyzer
ext install rust-lang.rust-analyzer

// CodeLLDB
ext install vadimcn.vscode-lldb

// crates
ext install serayuzgur.crates

// Even Better TOML
ext install tamasfe.even-better-toml
```

<br/>

```
^P
> Settings JSON
```

<br/>

```
"rust-analyzer.check.command": "clippy",
```

<br/>

### Приложение для примера

https://github.com/alfredodeza/rust-cli-example/tree/main
