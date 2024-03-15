# Using an SQL Database
When it comes to SQL, there's a lot of different database crates you can use depending on what you want.

These are split into mostly two different types, with there being quite a few different packages for using raw SQL with your database as well as an ORM. Note that while ORMs in Rust are still somewhat mostly as useful as they are in other languages, they can occasionally be a bit which reduces the advantage differential.

This section will talk about different libraries for usage with SQL.

## Raw SQL
There are a few libraries that see quite a lot of usage when we're talking about raw SQL:
- `tokio-postgres`/`rust-postgres`
- `sqlx`

`sqlx` is a raw SQL libraries that utilises the power of macros to be able to make the experience as ergonomic as possible. You can use the `sqlx::FromRow` macro to convert a `sqlx::Row` directly to your struct type (as long as it's declared). 

`tokio-postgres` on the other hand is a more low level library that also allows for SQL pipelining to bring you better querying speed. However, whether you actually need this benefit or not is mostly down to whether or not you actually need to write multiple queries at the same time. 

## ORMS/Query builders
There are a couple of crates that see quite a lot of usage when we're talking about query builders and ORMs:
- `sea-orm` 
- `diesel`

`sea-orm` is a full ORM that builds on `sqlx`, offering a much more rounded ORM experience that you might find in Ruby on Rails.

`diesel` while offering mostly ORM-like compatibilities, describes itself as more of a query builder.  

## What should I use?
Generally speaking, the library you should use depends on what you want:
- `SQLx` is somewhat slow in comparison to `tokio-postgres` but offers ease of use and compile-time safety.
- `tokio-postgres` is much faster than SQLx and is mostly easy to use, but unless you're using `cornucopia` you are losing out on the compile-time safety.
- `sea-orm` if you want the Ruby on Rails experience but in Rust and don't mind performance too much
- `diesel` if you want a query builder that is also very performant
