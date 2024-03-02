# Initialising a web service in Rust
To get started, you'll want to make sure that you have the Rust programming language installed. To do so, you can install it [here.](https://www.rust-lang.org/tools/install)

## Pre-written boilerplate
Here are some templates you can use to get started by skipping boilerplate with Rust:

- [Create-Rust-App](https://github.com/Wulf/create-rust-app) - A general full-stack template generator.
- [Shuttle Templates](https://github.com/shuttle-hq/shuttle-examples) - An easy way to get started with Rust and deploying to Shuttle, the Rust-native cloud platform.
- [Loco](loco.rs) - The Loco framework provides a SaaS (or "full-stack") starter that you can use.

## Tooling
Here are some tools you may want to install that you can use to become more productive. You can install them with `cargo install <package-name>`.
- [cargo-machete](https://github.com/bnjbvr/cargo-machete) - A package that will find unused dependencies and delete them from Cargo.toml for you.
- [cargo-sort](https://github.com/DevinR528/cargo-sort) - A package for sorting dependencies.
- [cargo-nextest](https://github.com/nextest-rs/nextest) - A superior test runner to the regular Rust test runner. 


## Popular Frameworks
- [Axum](https://github.com/tokio-rs/axum) - A framework that leverages the best parts of the Rust language tooling and has hyper-compatibility with the `hyper` and `tower` family of crates.
- [actix-web](https://github.com/actix/actix-web) - A battle tested framework that previously used the `actix` actor framework but now uses mostly `tokio` under the hood.
- [Rocket](https://github.com/rwf2/Rocket) - A framework that attempts to make it as easy as possible to write a web service by using macros.
- [Loco](https://github.com/loco-rs/loco) - A framework that leverages `sea_orm` and `axum` under the hood to provide a full-stack experience.
