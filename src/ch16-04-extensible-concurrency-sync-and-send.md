## Extensible Concurrency with the `Send` and `Sync` Traits

<!-- Old link, do not remove -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a>

Interestingly, almost every concurrency feature we’ve talked about so far in
this chapter has been part of the standard library, not the language. Your
options for handling concurrency are not limited to the language or the
standard library; you can write your own concurrency features or use those
written by others.

However, among the key concurrency concepts that are embedded in the language
rather than the standard library are the `std::marker` traits `Send` and `Sync`.

### Allowing Transference of Ownership Between Threads with `Send`

The `Send` marker trait indicates that ownership of values of the type
implementing `Send` can be transferred between threads. Almost every Rust type
implements `Send`, but there are some exceptions, including `Rc<T>`: this
cannot implement `Send` because if you cloned an `Rc<T>` value and tried to
transfer ownership of the clone to another thread, both threads might update
the reference count at the same time. For this reason, `Rc<T>` is implemented
for use in single-threaded situations where you don’t want to pay the
thread-safe performance penalty.

Therefore, Rust’s type system and trait bounds ensure that you can never
accidentally send an `Rc<T>` value across threads unsafely. When we tried to do
this in Listing 16-14, we got the error `` the trait `Send` is not implemented
for `Rc<Mutex<i32>>` ``. When we switched to `Arc<T>`, which does implement
`Send`, the code compiled.

Any type composed entirely of `Send` types is automatically marked as `Send` as
well. Almost all primitive types are `Send`, aside from raw pointers, which
we’ll discuss in Chapter 20.

### Allowing Access from Multiple Threads with `Sync`

The `Sync` marker trait indicates that it is safe for the type implementing
`Sync` to be referenced from multiple threads. In other words, any type `T`
implements `Sync` if `&T` (an immutable reference to `T`) implements `Send`,
meaning the reference can be sent safely to another thread. Similar to `Send`,
primitive types all implement `Sync`, and types composed entirely of types that
implement `Sync` also implement `Sync`.

The smart pointer `Rc<T>` also doesn’t implement `Sync` for the same reasons
that it doesn’t implement `Send`. The `RefCell<T>` type (which we talked about
in Chapter 15) and the family of related `Cell<T>` types don’t implement
`Sync`. The implementation of borrow checking that `RefCell<T>` does at runtime
is not thread-safe. The smart pointer `Mutex<T>` implements `Sync` and can be
used to share access with multiple threads, as you saw in [“Sharing a
`Mutex<T>` Between Multiple
Threads”][sharing-a-mutext-between-multiple-threads]<!-- ignore -->.

### Implementing `Send` and `Sync` Manually Is Unsafe

Because types composed entirely of other types that implement the `Send` and
`Sync` traits also automatically implement `Send` and `Sync`, we don’t have to
implement those traits manually. As marker traits, they don’t even have any
methods to implement. They’re just useful for enforcing invariants related to
concurrency.

Manually implementing these traits involves implementing unsafe Rust code.
We’ll talk about using unsafe Rust code in Chapter 20; for now, the important
information is that building new concurrent types not made up of `Send` and
`Sync` parts requires careful thought to uphold the safety guarantees. [“The
Rustonomicon”][nomicon] has more information about these guarantees and how to
uphold them.

## Summary

This isn’t the last you’ll see of concurrency in this book: the next chapter
focuses on async programming, and the project in Chapter 21 will use the
concepts in this chapter in a more realistic situation than the smaller
examples discussed here.

As mentioned earlier, because very little of how Rust handles concurrency is
part of the language, many concurrency solutions are implemented as crates.
These evolve more quickly than the standard library, so be sure to search
online for the current, state-of-the-art crates to use in multithreaded
situations.

The Rust standard library provides channels for message passing and smart
pointer types, such as `Mutex<T>` and `Arc<T>`, that are safe to use in
concurrent contexts. The type system and the borrow checker ensure that the
code using these solutions won’t end up with data races or invalid references.
Once you get your code to compile, you can rest assured that it will happily
run on multiple threads without the kinds of hard-to-track-down bugs common in
other languages. Concurrent programming is no longer a concept to be afraid of:
go forth and make your programs concurrent, fearlessly!

[sharing-a-mutext-between-multiple-threads]: ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads
[nomicon]: ../nomicon/index.html
