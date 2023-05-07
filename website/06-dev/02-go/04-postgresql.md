---
layout: page
title: Проверка возможности подключения к базе postgresql
description: Проверка возможности подключения к базе postgresql
keywords: Проверка возможности подключения к базе postgresql
permalink: /dev/go/postgresql/
---

# Проверка возможности подключения к базе postgresql

Подключаюсь к бесплатной базе облачного провайдера heroku.

<br/>

    $ go get github.com/lib/pq

<br/>

    $ vi main.go

<br/>

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/lib/pq"
)

const (
	host     = "ec2-23-23-184-76.compute-1.amazonaws.com"
	port     = 5432
	user     = "hrgcmhzjkgllyf"
	password = "f867d132e78e27e50a27d0b7522dbf3f44dc835c903eb3040d74ecd5daf5c633"
	dbname   = "d61hvpjfrp6em7"
	sslmode  = "require"
)

func main() {
	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=%s", host, port, user, password, dbname, sslmode)

	db, err := sql.Open("postgres", psqlInfo)
	if err != nil {
		panic(err)
	}

	err = db.Ping()
	if err != nil {
		panic(err)
	}

	fmt.Println("Successfully connectd!")
	db.Close()

}
```

Следует обратить внимание, требует ли база подключения по sslmode. Heroku требует.

<br/>

    $ go run main.go
    Successfully connectd!

<br/>

## Базовые примеры работы с базой postgresql на языке GO

Из видео курса "Learn to Create Web Applications using Go"

http://pgweb-demo.herokuapp.com/

```sql
CREATE TABLE users (
		id SERIAL PRIMARY KEY,
		name TEXT,
		email TEXT NOT NULL
);

CREATE TABLE orders (
		id SERIAL PRIMARY KEY,
		user_id INT NOT NULL,
		amount INT,
		description TEXT
)
```

<br/>

### INSERT ROW EXAMPE

```go

package main

import (
	"database/sql"
	"fmt"

	_ "github.com/lib/pq"
)

const (
	host     = "ec2-23-23-184-76.compute-1.amazonaws.com"
	port     = 5432
	user     = "hrgcmhzjkgllyf"
	password = "f867d132e78e27e50a27d0b7522dbf3f44dc835c903eb3040d74ecd5daf5c633"
	dbname   = "d61hvpjfrp6em7"
	sslmode  = "require"
)

func main() {
	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=%s", host, port, user, password, dbname, sslmode)

	db, err := sql.Open("postgres", psqlInfo)
	if err != nil {
		panic(err)
	}

	defer db.Close()

	var id int

	row := db.QueryRow(`
			INSERT INTO users(name, email)
			VALUES($1, $2)
			RETURNING id`,
		"Jon Clhoun", "jon@calhoun.io")
	err = row.Scan(&id)

	if err != nil {
		panic(err)
	}
	fmt.Println("id is ... ", id)

}


```

Еще

```go
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/lib/pq"
)

const (
	host     = "ec2-23-23-184-76.compute-1.amazonaws.com"
	port     = 5432
	user     = "hrgcmhzjkgllyf"
	password = "f867d132e78e27e50a27d0b7522dbf3f44dc835c903eb3040d74ecd5daf5c633"
	dbname   = "d61hvpjfrp6em7"
	sslmode  = "require"
)

func main() {
	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=%s", host, port, user, password, dbname, sslmode)

	db, err := sql.Open("postgres", psqlInfo)
	if err != nil {
		panic(err)
	}

	defer db.Close()

	for i := 1; i <= 6; i++ {
		userID := 1
		if i > 3 {
			userID = 3
		}

		amount := i * 100
		description := fmt.Sprintf("USB-C Adapter x%d", i)

		_, err = db.Exec(`
		INSERT INTO orders(user_id, amount, description)
		VALUES($1, $2, $3)`, userID, amount, description)

		if err != nil {
			panic(err)
		}

	}
}

```

<br/>

### SELECT 1 ROW

```sql

package main

import (
	"database/sql"
	"fmt"

	_ "github.com/lib/pq"
)

const (
	host     = "ec2-23-23-184-76.compute-1.amazonaws.com"
	port     = 5432
	user     = "hrgcmhzjkgllyf"
	password = "f867d132e78e27e50a27d0b7522dbf3f44dc835c903eb3040d74ecd5daf5c633"
	dbname   = "d61hvpjfrp6em7"
	sslmode  = "require"
)

func main() {
	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=%s", host, port, user, password, dbname, sslmode)

	db, err := sql.Open("postgres", psqlInfo)
	if err != nil {
		panic(err)
	}

	defer db.Close()

	var id int
	var name, email string

	row := db.QueryRow(`
			SELECT id, name, email
			FROM users
			WHERE id=$1`, 1)

	err = row.Scan(&id, &name, &email)

	if err != nil {
		if err == sql.ErrNoRows {
			fmt.Println("no rows")
		} else {
			panic(err)
		}

	}
	fmt.Println("id ", id, "name ", name, "email", email)

}
```

<br/>

### SELECT > 1 ROW

```sql
package main

import (
	"database/sql"
	"fmt"

	_ "github.com/lib/pq"
)

const (
	host     = "ec2-23-23-184-76.compute-1.amazonaws.com"
	port     = 5432
	user     = "hrgcmhzjkgllyf"
	password = "f867d132e78e27e50a27d0b7522dbf3f44dc835c903eb3040d74ecd5daf5c633"
	dbname   = "d61hvpjfrp6em7"
	sslmode  = "require"
)

func main() {
	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=%s", host, port, user, password, dbname, sslmode)

	db, err := sql.Open("postgres", psqlInfo)
	if err != nil {
		panic(err)
	}

	defer db.Close()

	type User struct {
		ID    int
		Name  string
		Email string
	}

	var users []User

	rows, err := db.Query(`
			SELECT id, name, email
			FROM users`)

	if err != nil {
		panic(err)
	}

	defer rows.Close()

	for rows.Next() {
		var user User
		err = rows.Scan(&user.ID, &user.Name, &user.Email)

		if err != nil {
			panic(err)
		}

		users = append(users, user)
	}

	if rows.Err() != nil {
		panic(rows.Err())
	}

	fmt.Println(users)

}

```

```go

package main

import (
	"database/sql"
	"fmt"

	_ "github.com/lib/pq"
)

const (
	host     = "ec2-23-23-184-76.compute-1.amazonaws.com"
	port     = 5432
	user     = "hrgcmhzjkgllyf"
	password = "f867d132e78e27e50a27d0b7522dbf3f44dc835c903eb3040d74ecd5daf5c633"
	dbname   = "d61hvpjfrp6em7"
	sslmode  = "require"
)

func main() {
	psqlInfo := fmt.Sprintf("host=%s port=%d user=%s password=%s dbname=%s sslmode=%s", host, port, user, password, dbname, sslmode)

	db, err := sql.Open("postgres", psqlInfo)
	if err != nil {
		panic(err)
	}

	defer db.Close()

	rows, err := db.Query(`
		SELECT * FROM users
		INNER JOIN orders ON users.id=orders.user_id`)

	if err != nil {
		panic(err)
	}

	for rows.Next() {
		var userID, orderID, amount int
		var email, name, desc string

		if err := rows.Scan(&userID, &name, &email, &orderID, &userID, &amount, &desc); err != nil {
			panic(err)
		}

		fmt.Println(": ", userID, ": ", name, ": ", email, ": ", orderID, ": ", userID, ": ", amount, ": ", desc)
	}

	if rows.Err() != nil {
		panic(rows.Err())
	}

}

```
