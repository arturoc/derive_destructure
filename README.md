# derive_destructure

This crate allows you to destructure structs that implement `Drop`.

If you've ever struggled with error E0509
"cannot move out of type `T`, which implements the `Drop` trait"
then this crate may be for you.

To use this crate, put this in your `lib.rs` or `main.rs`:
```rust
#[macro_use]
extern crate derive_destructure;
```

Then you have 2 ways to use this crate:

## Option 1: `#[derive(destructure)]`

If you mark a struct with `#[derive(destructure)]`, then you can destructure it using
```rust
let (field_1, field_2, ...) = my_struct.destructure();
```

This turns the struct into a tuple of its fields **without running the struct's `drop()`
method**. You can then happily move elements out of this tuple.

Note: in Rust, a tuple of 1 element is denoted as `(x,)`, not `(x)`.

## Option 2: `#[derive(remove_trait_impls)]`

If you mark your struct with `#[derive(remove_trait_impls)]`, then you can do
```rust
let my_struct = my_struct.remove_trait_impls();
```

The result is a struct with the same fields, but it implements no traits
(except automatically-implemented traits like `Sync` and `Send`).
In particular, it doesn't implement `Drop`, so you can move fields out of it.

The name of the resulting struct is the original name plus the suffix `WithoutTraitImpls`.
For example, `Foo` becomes `FooWithoutTraitImpls`. But you usually don't need to write
out this name.

`#[derive(remove_trait_impls)]` works on enums too.

## Example:
```rust
#[macro_use]
extern crate derive_destructure;

#[derive(destructure, remove_trait_impls)]
struct ImplementsDrop {
    some_str: String,
    some_int: i32
}

impl Drop for ImplementsDrop {
    fn drop(&mut self) {
        panic!("We don't want to drop this");
    }
}

fn main() {
    // Using destructure():
    let x = ImplementsDrop {
        some_str: "foo".to_owned(),
        some_int: 4
    };
    let (some_str, some_int) = x.destructure();
    // x's drop() method never gets called

    // Using remove_trait_impls():
    let x = ImplementsDrop {
        some_str: "foo".to_owned(),
        some_int: 4
    };
    let x = x.remove_trait_impls();
    // this x doesn't implement drop,
    // so we can move fields out of it
    drop(x.some_str);
    println!("{}", x.some_int);
}
```
