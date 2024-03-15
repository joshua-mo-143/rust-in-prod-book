# Rate Limiting
Rate limiting (or throttling) is a hugely vital component for any productionised web service, particularly if you want to prevent users from abusing your external-facing APIs (or someone tries to DDoS you!). 

## With governor
To get started with rate limiting, you'll want to use the `governor` crate which will allow you to rate limit your application.

Normal usage will allow you to create a rate limiter like this:
```rust
use std::num::NonZeroU32;
use nonzero_ext::*;
use governor::{Quota, RateLimiter};

let mut lim = RateLimiter::direct(Quota::per_second(nonzero!(50u32))); // Allow 50 units per second
assert_eq!(Ok(()), lim.check());
```
Then whenever you need to use the rate limiter, you use `.check()` to pass a request to the `RateLimiter`.

`governor` uses a GCRA (General Cell Rate Algorithm) which is like the leaky bucket method but more sophisticated.

`governor` additionally also has extra crates that you can use with your web application:
- `tower-governor` for Tower-based frameworks (Axum, Loco)
- `actix-governor` for actix-web
- `rocket-governor` for Rocket (although outdated at the time of writing)
