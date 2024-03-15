# Docker
This section will describe how you can use Docker to deploy your application either to a platform that lets you deploy via Dockerfiles (Fly.io, Railway, etc)

## The basics
Deploying with Docker is fairly easy via Rust. You can use a simple Dockerfile like this to package your web service and deploy it to an Alpine image:
```Dockerfile
FROM rust as builder
COPY . .
RUN cargo build --release 

FROM debian:bookworm-slim as runner
COPY --from=builder ./target/release/app /usr/local/bin

CMD = ["/usr/bin/app"]
```
Now you can deploy your Rust web service to anywhere that typically supports Dockerfiles!

## Speeding up your deployment
With Rust compilation times taking so long, typically it can take at least 4-5 minutes to deploy a simple Rust docker image. 

Thankfully, Luca Palmieri has given us an absolute treasure in the form of [cargo-chef](https://github.com/LukeMathWalker/cargo-chef), a Cargo plugin designed to be used in Dockerfiles to cache builds and speed up overall build times. 

A regular Dockerfile with `cargo-chef` looks like this:
```Dockerfile
FROM lukemathwalker/cargo-chef:latest-rust-1 AS chef
WORKDIR /app

FROM chef AS planner
COPY . .
RUN cargo chef prepare --recipe-path recipe.json

FROM chef AS builder 
COPY --from=planner /app/recipe.json recipe.json
# Build dependencies - this is the caching Docker layer!
RUN cargo chef cook --release --recipe-path recipe.json
# Build application
COPY . .
RUN cargo build --release --bin app

# We do not need the Rust toolchain to run the binary!
FROM debian:bookworm-slim AS runtime
WORKDIR /app
COPY --from=builder /app/target/release/app /usr/local/bin
ENTRYPOINT ["/usr/local/bin/app"]
```

You can also additionally run `cargo-chef` Dockerfiles with alpine as the final image. However, this requires you to build the `x86_64-unknown-linux-musl` target using `muslrust`. 

```Dockerfile
# Using the `rust-musl-builder` as base image, instead of 
# the official Rust toolchain
FROM clux/muslrust:stable AS chef
USER root
RUN cargo install cargo-chef
WORKDIR /app

FROM chef AS planner
COPY . .
RUN cargo chef prepare --recipe-path recipe.json

FROM chef AS builder
COPY --from=planner /app/recipe.json recipe.json
# Notice that we are specifying the --target flag!
RUN cargo chef cook --release --target x86_64-unknown-linux-musl --recipe-path recipe.json
COPY . .
RUN cargo build --release --target x86_64-unknown-linux-musl --bin app

FROM alpine AS runtime
RUN addgroup -S myuser && adduser -S myuser -G myuser
COPY --from=builder /app/target/x86_64-unknown-linux-musl/release/app /usr/local/bin/
USER myuser
CMD ["/usr/local/bin/app"]
```
