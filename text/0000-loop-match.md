- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: (leave this empty)
- Rust Issue: (leave this empty)

# Summary

Complete the 'patternification' of Rust's control structures by adding `loop match`, `for match`, and `else match` to the language.

# Motivation

A common idiom in Rust is to consume values out of a channel or stream into a match, in a `loop`:

    loop {
        match rx.recv() {
            Foo(x) => ...
            Bar(a,b) => ...
            ...
        }
    }

A similar idiom is also common in `for` loops:

    for i in foo.iter() {
        match i {
            Foo(x) => ...
            Bar(a,b) => ...
            ...
        }
    }

You will also occasionally see `else` blocks using a similar pattern:

    if blah {
        ...
    } else {
        match rx.recv() {
            Foo(x) => ...
            Bar(a, b) => ...
            ...
        }
    }

When there's no additional logic in the block outside of the match, the extra pair of braces, two vertical lines, and indentation are visual clutter with no offsetting benefit, so it would be nice to remove them.


# Detailed design

With `loop match`, the code from above would become:

    loop match rx.recv() {
        Foo(x) => ...
        Bar(a,b) => ...
        ...
    }

With `for match`, the code from above would become:

    for match i in foo.iter() {
        Foo(x) => ...
        Bar(a, b) => ...
        ...
    }

With `else match`, the code from above would become:

    if blah {
        ...
    } else match rx.recv() {
        Foo(x) => ...
        Bar(a, b) => ...
        ...
    }


`loop match`, `for match`, and `else match` can be seen as the analogous pattern-handling extensions to `loop`, `for`, and `else` that `if let` and `while let` were for `if` and `while`.


# Drawbacks

- Adds more features to the language.

- A very common variation on the basic idiom for `for` loops is to match on an expression involving the loop variable.  This doesn't benefit from this rfc:

        for i in foo.iter() {
            match i.stuff() {
                Foo(x) => ...
                Bar(a,b) => ...
                ...
            }
        }


# Alternatives

- Do nothing.

- Instead of brand-new syntax, the vertical and horizontal space complaints could be addressed by adopting

        loop { match rx.recv() {
            Foo(x) => ...
            Bar(a,b) => ...
            ...
        }}

    as idiomatic code.  However, this still requires an extra pair of braces, which makes it a little ugly.

- Don't require the 'match' keyword.  This would be trivially unambiguous for `loop` and `else`,
because their current forms don't take a param, while their match forms would:

        loop rx.recv() {
            Foo(x) => ...
            Bar(a,b) => ...
            ...
        }

        if blah {
            ...
        } else rx.recv() {
            Foo(x) => ...
            Bar(a, b) => ...
            ...
        }

    Since '=>' is unique to pattern matching in the language, `for` could also join the no-`match` party, but it would require more lookahead:

        for i in foo.iter() {
            Foo(x) => ...
            Bar(a,b) => ...
            ...
        }



# Unresolved questions

I can't speak authoritatively to implementation details.  These changes seem like they would be in the same ballpark as `if let` in terms of complexity, but someone with more experience in the compiler would need to weigh in.
