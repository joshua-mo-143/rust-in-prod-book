# Shuttle
Shuttle is a Rust-native cloud dev platform that lets you deploy in one line. Using their runtime, you can quickly provision resources that can be spun up both locally and in production without issue. 

## Shuttle Project anatomy
Shuttle projects are just like regular Rust projects with one major difference: they use the Shuttle runtime. Your main function should look like this:
```rust
use axum::{Router, routing::get};

#[shuttle_runtime::main]
async fn my_fn() -> shuttle_axum::Axum {
    let router = Router::new();
    
    Ok(router.into())
}
```

You can use `cargo shuttle deploy` (with the `--ad` flag if on a dirty Git branch) to deploy.

## Shuttle Projects in a workspace
Currently when using a Shuttle project in a workspace, it is required to be the primary binary file for execution. This is to say that you will probably be using `cargo shuttle run` to run your application most of the time - or otherwise you'll end up using `cargo run --bin <alt-binary-name>` to be able to do this.

## I don't want to use a web framework / I want to run my own program!
`shuttle_runtime` is essentially able to run anything you want (as long as it's HTTP-based) by exposing a trait called `Service`. By using this trait, you can do things like:
- Run multiple tokio tasks at the same time
- Run a non-official Shuttle integration (for example a Slack bot)
- Run a Discord bot and Axum router in the same tokio task using `tokio::join!()`

The trait looks like this in usage:
```rust
use axum::{Router, routing::get};
use std::net::SocketAddr;
use tokio::net::TcpListener;

struct MyStruct {};

#[shuttle_runtime::main]
async fn my_fn() -> Result<MyStruct, shuttle_runtime::Error> {
    Ok( MyStruct {} )
}

impl shuttle_runtime::Service for MyStruct {
    async fn bind(mut self, addr: SocketAddr) -> Result<(), shuttle_runtime::Error> {
       let router = Router::new();
       
       let tcp_listener = TcpListener::bind(&addr).await.unwrap();
       
       axum::serve(tcp_listener, router).await.unwrap();
       
       Ok(())
    }
}
```
As long as the task you are trying to run can be run asynchronously in a loop, this works.

## Advantages of using Shuttle
- Easy deployment - no infra config required (it gets set up for you)
- Generous free plan
- Bleeding edge of Rust

## Disadvantages of using Shuttle
- Bleeding edge means that the platform can have breaking changes fairly often
- Currently only HTTP bound services are possible (TBC in future)
- Using Shuttle in a workspace can be a bit finicky on occasion
