---
layout: page
title: Пример компиляции приложения Golang
description: Пример компиляции приложения Golang
keywords: dev, golang, linux, run
permalink: /dev/go/run/
---

# Пример компиляции приложения Golang

```
$ cd ~/tmp/
$ vi ./main.go
```

<br/>

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, "Hello, world!")
	})

	http.ListenAndServe(":3000", nil)
}
```

<br/>

```
$ curl localhost:3000
Hello, world!
```

<br/>

```
^C
```

<br/>

```
$ go build ./main.go
$ ./main
```

<br/>

```
$ curl localhost:3000
Hello, world!
```
