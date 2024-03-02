# tokio-postgres
`tokio-postgres` is a low level crate designed to be as fast as possible by allowing SQL query pipelining. 

## Getting Started
To get started, you can add `tokio-postgres` to your crate by running the following:
```rust
cargo add tokio-postgres
```

You can find out more about how to implement support for things like `serde_json` and `chrono` by following the link [here](https://docs.rs/tokio-postgres/latest/tokio_postgres/#features).  

## Usage
Basic usage can be seen below (from the docs):
```rust
use tokio_postgres::{NoTls, Error};

#[tokio::main] // By default, tokio_postgres uses the tokio crate as its runtime.
async fn main() -> Result<(), Error> {
    // Connect to the database.
    let (client, connection) =
        tokio_postgres::connect("host=localhost user=postgres", NoTls).await?;

    // The connection object performs the actual communication with the database,
    // so spawn it off to run on its own.
    tokio::spawn(async move {
        if let Err(e) = connection.await {
            eprintln!("connection error: {}", e);
        }
    });

    // Now we can execute a simple statement that just returns its parameter.
    let rows = client
        .query("SELECT $1::TEXT", &[&"hello world"])
        .await?;

    // And then check that we got back the same string we sent over.
    let value: &str = rows[0].get(0);
    assert_eq!(value, "hello world");

    Ok(())
}
```

Note that the outcome of a statement will always be a `tokio_postgres::Row`. There is currently no way (with **only** usage of `tokio-postgres`) to automatically turn a query into a struct like with SQLx.

## Pipeline Querying
As mentioned in the first paragraph, `tokio-postgres` supports SQL pipeline queries. This means that queries can be processed *and* returned concurrently, which is a significant performance improvement.

You can see how this works below - essentially, all of the queries are joined up into a single Future.
```rust
use futures_util::future;
use std::future::Future;
use tokio_postgres::{Client, Error, Statement};

async fn pipelined_prepare(
    client: &Client,
) -> Result<(Statement, Statement), Error>
{
    future::try_join(
        client.prepare("SELECT * FROM foo"),
        client.prepare("INSERT INTO bar (id, name) VALUES ($1, $2)")
    ).await
}
```

Pipelining happens automatically whenever two or more futures have been joined together - you don't need to worry about handling any of this manually.

## Cornucopia
`cornucopia` is a CLI tool you can use to generate type-safe queries for your queries. It mostly acts as a thin layer over the library and when run, will generate types from your raw SQL queries (including structs!).

## Usage
When defining your SQL queries, you can do so like this using the `--` syntax. Note that you only need to define the nullable columns - non-nullable columns will already be defined for you when generating the Rust glue code for you. There's a couple of things going on here:
- When using `--:`, you are defining a struct
- When using `--!`, you are defining a query where it follows the key `function_name (nullable_items?) : ReturnedStruct`.
```sql
--: Author(age?)

--! authors : Author
SELECT name, age FROM Authors;

--! authors_from_country (country?) : Author
SELECT name, age
FROM Authors
WHERE Authors.nationality = :country;
``` 

When generated, the code will live in a file called `cornucopia.rs` in the top level of your `src` folder. 

## Pipelining 

