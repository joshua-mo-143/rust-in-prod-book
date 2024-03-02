# Error Handling
Error handling is an extremely useful component of Rust that allows us to operate much faster when we've set up all of the error propagation. Let's have a quick look at an enum that represents a couple of different ways an error can occur:
```rust
#[derive(Debug)]
pub enum MyError {
    StdIoError(std::io::Error),
    SQLError(sqlx::Error),
    NoDataFound(sqlx::Error)
}
```

To be able to return this as an error type without any assistance, we need to implement `std::error::Error` for the type:

```rust
impl std::error::Error for MyError {}
```

This now allows us to use it as an error type in our functions!

However, we can go one step further. Let's try implementing `From<std::io::Error>` for `MyError` which will allow us to automatically propagate the error up the call stack.
```rust
impl From<std::io::Error> for MyError {
    fn from(err: std::io::Error) -> Self {
        Self::StdIoError(err)
    }
}
```
Note that we are using the full export path here. While it's almost certainly more convenient to import the error at the top of the file, you may want to keep the full path here for absolute transparency on what you're implementing.

Now when using any function or method that can return `std::io::Error` as an error type, we can use the `?` operator to be able to automatically attempt to grab the inner variable from a `Result<T, E>`; if the result variant is an error, the function will automatically return an error. 

Now we can go from this:
```
use std::fs::File;

fn my_function() -> Result<(), std::io::Error> {
    let file_reader: String = fs::read_to_string("address.txt")?.parse()?;
}

```
To this:
```
use std::fs::File;

fn my_function() -> Result<(), MyError> {
    let foo: String = fs::read_to_string("address.txt")?.parse()?;
}
```
While there isn't much difference visually, we can now implement `From<T>` for all of our other enum variants and now we can use `MyError` for easy error propagation in any function that can return the errors!

Note that the only condition required to be able to use `?` is that the function signature error type must implement `From<T>` so that the type can be turned into the upper-level error type, or it must be the same type as the function signature error type.

However, if you have a lot of different enum variants this can take quite a while to write all the `From<T>` implementations. Instead, we can use macros from [`thiserror`](https://github.com/dtolnay/thiserror) to automatically do all of this for us!

This example from the `thiserror` GitHub repo shows this example for basic usage:
```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum DataStoreError {
    #[error("data store disconnected")]
    Disconnect(#[from] io::Error),
    #[error("the data for key `{0}` is not available")]
    Redaction(String),
    #[error("invalid header (expected {expected:?}, found {found:?})")]
    InvalidHeader {
        expected: String,
        found: String,
    },
    #[error("unknown data store error")]
    Unknown,
}
```
There are a few things going on here:
- Usage of the `thiserror::Error` derive macro that allows the magic to happen
- The `#[from]` attribute that lets us automatically derive `From<T>`
- The `#[error("..")]` attribute that automatically derives the `std::fmt::Display` implementation (and therefore. `.to_string()`)

If you want more complex error handling however, you will still want to use `From<T>` manually. Take this `From<sqlx::Error>` snippet for example:
```rust
impl From<sqlx::Error> for MyError {
    fn from(err: sqlx::Error) -> Self {
        match self {
            sqlx::Error::RowNotFound => Self::NoDataFound(err),
            _ => Self::SQLError(err),
            } 
        }
    }
}
```

Let's say we have a web service on Axum that serves some data from Postgres. If we make a request to the service but there's no data there, we can do things like returning a `NO_CONTENT` HTTP header instead of an error. This makes debugging much easier to handle (and read!), which is all the better for us. 
