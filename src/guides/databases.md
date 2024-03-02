# Using an SQL Database
When it comes to SQL, there's a lot of different database crates you can use depending on what you want.

These are split into mostly two different types, with there being quite a few different packages for using raw SQL with your database as well as an ORM. Note that while ORMs in Rust are still somewhat mostly as useful as they are in other languages, they can occasionally be a bit which reduces the advantage differential.

This section will talk about different libraries for usage with SQL.

## Raw SQL
There are a few libraries that see quite a lot of usage when we're talking about query builders:
- `tokio-postgres`
- `sqlx`

Each of these libraries will have their own section. Stay tuned! For now, we will talk about the primary differences.

`sqlx` is a raw SQL libraries that utilises the power of macros to be able to make the experience as ergonomic as possible. You can use the `sqlx::FromRow` macro to convert a `sqlx::Row` directly to your struct type (as long as it's declared). 

`tokio-postgres` on the other hand is a more low level library that also allows for SQL pipelining to bring you better querying speed. However, whether you actually need this benefit or not is mostly down to whether or not you actually need to write multiple queries at the same time. 

## A side-note on MSSQL
While you're browsing Crates.io for SQL libraries, you will probably note that Microsoft SQL has very little support when it comes to Rust. This is partially due to the fact that proprietary software is very difficult to test outside of being in a company that actually has licenses to use MSSQL. However, there is a way you can use MSSQL through Tiberius, the Rust MSSQL driver by Prisma. 

