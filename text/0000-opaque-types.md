- Feature Name: Opauque Type Aliases
- Start Date: 2015-07-01
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

The current state of type aliases do not account for Rust's FFI utility. Opaque
type alises would help with the orgonomics of these features.

# Motivation

The current state of rustc_llvm is unfortunate. Many unrelated params are
handed around as `ValueRef`s when they could be hidden behind opaque types in
rust land, preventing cross contamination.

# Detailed design

The proposal is to introduce a new keyword; `opaque`. An opaque type behaves
like an existing type alias, insofar as it mimics an existing defined type.
However, instances of that type will not fulfil requirements for the opaque
type, requiring conversion.

To make this possible, opaque types will provide a constructor, such that:

```rust
opaque type T = u64;

extern "C" {
    fn takes_T(t: T);
}

fn main() {
    let t: T = T(124);
    takes_T(t);
}
```


# Drawbacks

Introducing a new keyword is not to be taken lightly.

// TODO: Grep crates.io for uses of `opaque` as an ident.

# Alternatives

The alternative is to use unit structs, which require un-ergonomic
desctructuring for use, and are prohibitively costly in FFI contexts.

# Unresolved questions

The design itself is still in flux. While I've considered at a high level the
problem I'm trying to solve, there are undoubtedly issues I have not
anticipated
