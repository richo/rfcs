- Feature Name: Opauque Type Aliases
- Start Date: 2015-07-01
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

The current state of type aliases do not account for Rust's FFI utility. Opaque
type alises would help with the orgonomics of these features.

# Motivation

One of rust's key strengths is it's interoperability with existing programs
that use calling conventions understood by LLVM. However, in today's world
there is considerable type erasure across interface boundaries, as primitive
types are interchangeable.

Thus, library that `typedef`s several types of integers, or pointer types is
prone to mistakes in rust land, as using using "the wrong" pointer or primitive
type will not upset typeck.

As a concrete example, much of rustc_llvm uses raw pointers to zero variant

# Detailed design

The proposal is to introduce a new keyword; `opaque`. An opaque type alias
behaves the same way as Golang's type aliases- types are inferred for literals,
but values of a concrete, incompatible type are disallowed.

To make this possible, the existing `as` coercions will continue to work as expected:

```rust
opaque type Size = usize;
opaque type Pointer = *const Size;

#[link(name="c_lib")]
extern "C" {
    fn takes_Size(s: Size);
    fn takes_Pointer(ptr: Pointer);
}

// Glue
fn Size(s: usize) -> Size {
    s as Size
}

fn main() { unsafe {
    let size: Size = Size(124);
    // Ok
    takes_Size(size);
    // Ok
    takes_Size(0x100);

    // Ok
    takes_Pointer(&size);

    let value: usize = 0x125;
    // Not ok
    takes_Size(value);
    //  ok
    takes_Pointer(&value);

    // Ok
    takes_Pointer(&0x70);
}}
```

This design offers a lighter weight alternative to single member tuple structs
for native rust code in which implementation details ought to be hidden, and is
necessary across FFI boundaries.


# Drawbacks

Introducing a new keyword is unfortunate, however grepping my cargo directory I
can find only a few libraries currently using `opaque` in an ident position.

The libraries using `opaque` are:
* miniz-sys
* libz-sys
* rust-cocoa

# Alternatives

The alternative is to use unit structs, which require un-ergonomic
desctructuring for use, and are prohibitively costly in FFI contexts.

Alternatively, this could be accomplished with an #[opaque] attr on type
aliases, which would be completely backward compatible.

# Unresolved questions

None immediately obvious to me.
