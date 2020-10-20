# Copy type ergonomics

## Background

There are a number of pain points with `Copy` types that the lang team is
interested in exploring, though active experimentation is not currently ongoing.

Some key problems are:

## `Copy` cannot be implemented with non-`Copy` members

There are standard library types where the lack of a `Copy` impl is an
active pain point, e.g., [`MaybeUninit`](https://github.com/rust-lang/rust/issues/62835)
and [`UnsafeCell`](https://github.com/rust-lang/rust/issues/25053)

### History

 * `unsafe impl Copy for T` which avoids the requirement that T is recursively
   Copy, but is obviously unsafe.
    * https://github.com/rust-lang/rust/issues/25053#issuecomment-218610508
 * `Copy` is dangerous on types like `UnsafeCell` where `&UnsafeCell<T>`
   otherwise would not permit access to `T` in [safe
   code](https://github.com/rust-lang/rust/issues/25053#issuecomment-98447164).

## `Copy` types can be (unintentionally) copied

Even if a type is Copy (e.g., `[u8; 1024]`) it may not be a good idea to make
use of that in practice, since copying large amounts of data is slow.

This also comes up with `Copy` impls on `Range`, which would generally be
desirable but is error-prone given the `Iterator/IntoIterator` impls on ranges.

Note that "large copies" comes up with moves as well (whih are copies, just
taking ownership as well), so a size-based lint is plausibly desirable for both.

### History

* Tracking issue: [#45683](https://github.com/rust-lang/rust/issues/45683)

## References to `Copy` types

Frequently when dealing with code generic over T you end up needing things like
`[u8]::contains(&5)` which is ugly and annoying. Iterators of copy types also
produce `&&u64` and similar constructs which can produce unexpected type errors.

There is also plausibly performance left on the table with types like `&&u64`.

Note that this interacts with the unintentional copies (especially of large
structures).

This could plausibly be done with moved values as well, so long as the
semantics match the syntax (e.g. `wants_ref(foo)` acts like `wants_ref(&{foo})`)
similar to how one can pass `&mut` to something that only wants `&`.

### History

* [RFC 2111 (not merged)](https://github.com/rust-lang/rfcs/pull/2111)
* [Rust tracking issue (closed)](https://github.com/rust-lang/rust/issues/44763)
* "Allow owned values where references are expected" in [rust-roadmap-2017#17](https://github.com/rust-lang/rust-roadmap-2017/issues/17)
