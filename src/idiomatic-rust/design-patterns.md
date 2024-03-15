# Design Patterns
This section will give you a run-down on what Rust patterns are universally useful when building applications. No matter what kind of application you're building, these will almost certainly help you.

## The Builder Pattern
The Builder pattern is a pattern that looks something like this, where all of the fields that require user input are labelled as `Option<T>` in the builder:

```rust
pub struct MyStruct {
    id: i32,
    note: String,
}

pub struct MyStructBuilder {
    id: i32,
    note: Option<String>,
}
```

Then you might have an impl that looks like this:

```rust
impl MyStructBuilder {
    fn new() -> Self {
        Self {
         id: 1,
         note: None
         }
    }

    fn note(&mut self, note: String) {
        self.note = Some(note);
    }

    fn build(self) -> Result<MyStruct, String> {
        MyStruct {
           id: self.id,
           note: let Some(note) = self.note else {
               return Err("There's no note here!".to_string())
           }
        }
    }
}

```
What are the benefits of this?
- Depending on your use case, you can have a builder that is able to build into several different kinds of structs depending on the inputs.
- Makes the code cleaner when having to implement builder methods for a large struct that has defaults
