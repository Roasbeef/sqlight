# sqlight_loom

This is a packaging fork of [sqlight 1.2.0](https://github.com/lpil/sqlight).
The Gleam `sqlight` module and its API are unchanged. The Hex package is named
`sqlight_loom` and selects `esqlite_loom` 0.9.0, which retires private query
statements before returning. Depend on this package instead of `sqlight`;
both packages define the same modules and cannot be used together.

```toml
[dependencies]
sqlight_loom = "== 1.2.0"
```

```gleam
import sqlight
```

The binding source is unchanged from upstream commit
`b19f58d9f1543b9cf7efd3da8b00e09900f2dd08`. Stock Gleam builds this package;
Rebar3 and a C compiler build its native SQLite dependency.

The original sqlight documentation follows.

---

# sqlight

[![Package Version](https://img.shields.io/hexpm/v/sqlight)](https://hex.pm/packages/sqlight)
[![Hex Docs](https://img.shields.io/badge/hex-docs-ffaff3)](https://hexdocs.pm/sqlight/)

Use [SQLite](https://www.sqlite.org/index.html) from Gleam!

Works on Erlang and on JavaScript runtimes that support `node:sqlite`, like
NodeJS, and NodeJS.

```sh
gleam add sqlight
```

```gleam
import gleam/dynamic/decode
import sqlight

pub fn main() {
  use conn <- sqlight.with_connection(":memory:")

  let sql = "
  create table cats (name text, age int);

  insert into cats (name, age) values 
  ('Nubi', 4),
  ('Biffy', 10),
  ('Ginny', 6);
  "
  let assert Ok(Nil) = sqlight.exec(sql, conn)

  let cat_decoder = {
    use name <- decode.field(0, decode.string)
    use age <- decode.field(1, decode.int)
    decode.success(#(name, age))
  }

  let sql = "
  select name, age from cats
  where age < ?
  "
  let assert Ok([#("Nubi", 4), #("Ginny", 6)]) =
    sqlight.query(sql, on: conn, with: [sqlight.int(7)], expecting: cat_decoder)
}
```

Documentation can be found at <https://hexdocs.pm/sqlight>.

## Why SQLite?

SQLite is a implementation of SQL as a library. This means that you don't run a
separate SQL server that your program communicates with, but you embed the SQL
implementation directly in your program. SQLite stores its data in a single
file. The file format is portable between different machine architectures. It
supports atomic transactions and it is possible to access the file by multiple
processes and different programs.

You can also use in-memory databases with SQLite, which may be useful for testing.

## Implementation

When running on Erlang it is a library wrapper around the excellent Erlang library
[esqlite](https://hex.pm/packages/esqlite), which in turn is a wrapper around
the SQLite C library. It is implemented as a NIF, which means that the SQLite
database engine is linked to the erlang virtual machine.

When running on JavaScript it is a wrapper around the
[`node:sqlite`](https://nodejs.org/api/sqlite.html) module that is built-in to
the most common runtimes.

## On using Bool with SQLite

SQLite does not have a native boolean type. Instead, it uses ints, where 0 is
False and 1 is True. Because of this the Gleam stdlib decoder for bools will not
work, instead the `sqlight.decode_bool` function should be used as it supports
both ints and bools.
