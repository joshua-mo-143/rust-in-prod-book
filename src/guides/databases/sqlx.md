# SQLx
`sqlx` (stylized as "SQLx") is a raw SQL library that additionally uses macros to allow you to ergonomically leverage the Rust tooling.

## Getting Started
Note that to be able to run SQLx (like many other async Rust database libraries), you need to enable two different types of features:
- An SQL backend (Postgres, MySQL, etc)
- A runtime (in this case, we'll be using the Tokio rustls runtime)

Runtimes offer both native SSL and rustls versions, depending on what you're interested in using.

You can install it like so:
```rust
cargo add sqlx -F postgres,runtime-tokio-rustls
```
SQLx also has a CLI you can additionally use called `sqlx-cli` to make your life easier. You can install it with the following shell snippet:
```bash
cargo install sqlx-cli
```
By default, it will install all of the features for you. This may be useful if you're operating over all kinds of SQL databases (that SQLx supports). If you only need to operate over one type of database (Postgres for example) you can add the feature specifically like so:
```rust
cargo install sqlx-cli --no-default-features --features rustls,postgres
```

## Querying
To get started, we can create a simple `Pool<Postgres>` like so:
```rust
use sqlx::PgPool;

async fn connect_to_db() -> PgPool, sqlx::Error> {
    let pool = PgPool::connect("postgres://postgres:postgres@localhost:5432/postgres")
        .await?;
    
    Ok(pool)
}
```
Now we can query it like so:
```rust
async fn do_a_query() -> Result<(), sqlx::Error> {
   let pool = connect_to_db();
   
   sqlx::query("SELECT 'HELLO WORLD!'")
       .execute(&pool)
       .await?;
       
   Ok(())
}
```
SQLx offers a few types of queries:
- Regular queries where you use `row.get()` to retrieve the row 
- `query_as` queries that auto-return a declared Struct that implements `sqlx::FromRow`
  - This requires the `macros` feature (see below)
- Scalar queries where the result gets returned as a Tuple 
- Queries that get read in from a file

## SQLx macros
There are two primary use cases for using the SQLx macros:
- Using the compile-time safety macros to ensure type safety (requires `sqlx-cli` installed)
- Using the derive macros to add extra attributes to your structs

### Derive macros
SQLx macros make up a significant chunk of the library utility and can be hugely effective depending on what you're trying to do. You can add it like so:
```rust
cargo add sqlx -F macros
```
You can also add it manually in your Cargo.toml file.

The main macro being used here is primarily `sqlx::FromRow` - a derive macro for structs that allows them to be auto-returned by `query_as!()` (or `query\_as::<_, T>`, depending on what your needs are. It looks like this:
```rust
use sqlx::FromRow;

#[derive(FromRow, Debug)]
struct MyStruct {
    id: i32,
    message: String,
}
```
Now when we're querying our struct, we can simply use `query_as()` instead of `query()` to be able to get the value out:
```rust

fn get_my_struct() -> Result<(), sqlx::Error> {
    let pool = connect_to_db();
    
    let mystruct = sqlx::query_as::<_, MyStruct>("SELECT id, message FROM table")
        .fetch_all(&pool)
        .await?;
        
    println!("{:?}", mystruct);
    
    Ok(())
}
```
There are also additionally other things you can do with SQLx like adding attributes. You can add an attribute to attempt to turn into a type from another type, for example:
```rust
#[derive(FromRow, Debug)]
struct MyStruct {
    id: i32,
    message: String,
    #[sqlx(try_from = "i32")]
    points: u32
}
```
In order to do this however, your type needs to implement `From<T>` for the target type. There are some cases in which this is not strictly possible however, so you will need to wrap the type in a struct to create a newtype that then can be extended by your application. 

### Compile-time safety macros
Firstly, we'll talk about macros in SQLx that make your queries typesafe. Note that you will need `sqlx-cli` installed to make full use of this.

All of the different ways of querying in SQLx have a macro equivalent (`query!()`, `query_as!()`, `query_scalar!()`, `query_file!()`). You just need to add them to your code with the correct types, run `cargo sqlx prepare` and it will work.

See below for a basic example of usage:
```rust
#[derive(FromRow)]
struct MyStruct {
    message: String
}

async fn do_a_query() -> Result<(), sqlx::Error> {
   let pool = connect_to_db();
   
   let res = sqlx::query!("SELECT 'HELLO WORLD!'")
       .fetch_one(&pool)
       .await?;
       
   let res = sqlx::query_as!(MyStruct, "SELECT message from table where id = $1", 1)
       .fetch_one(&pool)
       .await?;
       
   Ok(())
}
```
Once you've added all the query macros you need, you can then use `cargo sqlx prepare` to be able to run everything.

## Nested structs
Currently with regards to nested structs, there's a couple of ways we can do this:
- Use `#[sqlx(flatten)]` on a given field. 
- Use `JSON_AGG` to aggregate all of your fields into a `serde_json::Value` type, retrieve that from the database (requires `json` feature enabled) and then use `serde_json::from_str` to do it.
- Re-implement `sqlx::FromRow` manually for your database.

Which one you need to use basically depends on your use case:
- `#[sqlx(flatten)]` is typically best for when all the values are in one table and don't have conflicting names
- `JSON_AGG` is mainly useful for if you are using things like `LEFT JOIN LATERAL` - however, note that the conversion to JSON and back does incur a performance overhead.
- Manual `sqlx::FromRow` adds the most boilerplate but allows the most low-level control over the decoding. Note that you'll also want to implement `sqlx::FromRow` if you need custom decoding behavior for a struct (for example, discarding certain values).

## Create your own SQLx-compatible type
Although you can use `sqlx::FromRow` for most things, sometimes you may want to create a struct with Rust types that are not natively supported by SQLx. To do this, you'll need to implement `sqlx::Decode` and `sqlx::Encode` to be able to do this.

Although the SQLx trait documentation is a little bit complicated, we can learn how it works by looking at the source code. The implementation for the `String` type looks like this: 
```rust
impl Encode<'_, MySql> for String {
    fn encode_by_ref(&self, buf: &mut Vec<u8>) -> IsNull {
        <&str as Encode<MySql>>::encode(&**self, buf)
    }
}

impl Decode<'_, MySql> for String {
    fn decode(value: MySqlValueRef<'_>) -> Result<Self, BoxDynError> {
        <&str as Decode<MySql>>::decode(value).map(ToOwned::to_owned)
    }
}
```
What can we learn from this?
- The first part of the `Encode<'_, _>_` is simply for "any lifetime" and does not matter hugely.
- The second generic is the database backend that we're using. For example, if we were using Postgres, it would look like this:
```rust
impl Encode<'_, Postgres> for String {
    fn encode_by_ref(&self, buf: &mut Vec<u8>) -> IsNull {
        <&str as Encode<Postgres>>::encode(&**self, buf)
    }
}

impl Decode<'_, Postgres> for String {
    fn decode(value: PostgresValueRef<'_>) -> Result<Self, BoxDynError> {
        <&str as Decode<Postgres>>::decode(value).map(ToOwned::to_owned)
    }
}
```
The most simple way to go about this would be converting our known type into a Postgres-compatible type (for example, `String` or `u32`/`i32`). If you want to implement customised behaviour for a type that is already natively supported by SQLx, you will want to wrap it in a newtype struct.
