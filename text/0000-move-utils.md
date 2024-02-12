- Start Date: (fill me in with today's date, YYYY-MM-DD)
- RFC PR: [amaranth-lang/rfcs#0000](https://github.com/amaranth-lang/rfcs/pull/0000)
- Amaranth Issue: [amaranth-lang/amaranth#0000](https://github.com/amaranth-lang/amaranth/issues/0000)

# Move `utils` out of toplevel

## Summary
[summary]: #summary

Rename `amaranth.utils`to `amaranth.lib.math`.

## Motivation
[motivation]: #motivation

This came up as part of a project to fill gaps in the Amaranth documentation. In this project, we have been organizing all Amaranth functionality into two categories:

- The "language", living in `amaranth.hdl` and exposed either by the `from amaranth import *` prelude or by directly importing from `amaranth.hdl`. These are documented in the Language Guide and Language Reference.
- The "standard library", living in `amaranth.lib` and documented in the Standard Library section of the website.

In Amaranth, `utils.py` contains a small number of functions related to discrete math and register sizes. This file originally existed as the container for two functions named `log2_int` and `bits_for` which at the time "didn't go anywhere else"; it is one of the oldest pieces of code in Amaranth and predates all current schemes of organization. As such it is accessed through `amaranth.utils`, which violates the organization scheme. As part of the documentation project we would like to document `utils`, but this is difficult becuase its placement makes it unclear if it is part of the language, part of the standard library, or what.

We can resolve this confusion by moving the `utils` functions into a new top-level item in the standard library, and I propose `math`.

## Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

The new `math` module should be in "category 3" in the standard library documentation, and its top level summary should be:

The `amaranth.lib.math` module provides math functions which are useful in circuit design, but not part of the Python standard library. The most frequently useful of these functions is `bits_for`, which can be used to create a Shape from a sample value:

	from amaranth.lib.math import bits_for

    MAX_VALUE = 386
    # An unsigned representing numbers from 0 to MAX_VALUE inclusive
    shape = bits_for(MAX_VALUE)
    signal = Signal(shape)

## Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

Beneath this top level summary should be full docs for the `ceil_log2`, `exact_log2` and `bits_for` functions.

## Drawbacks
[drawbacks]: #drawbacks

Although utils has never been documented, it is widely used. This has already been a site of recent RFCs (see [0017](0017-remove-log2-int.md)) renaming functions, so further changes will be irritating to existing users.

## Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

The goal here is to make the code "documentable"; I.E., to make the code rational enough we can explain it to users. There are different ways we could do this; the author of this RFC believes they are all equivalent, as long as we can clearly explain why we made the decision we made.

**Should `utils` go in the language or the standard library?**

In general, the rule for what is "language" and what is "standard library" is that if something *could* be in the standard library, it is. The [introductory docs](https://amaranth-lang.org/docs/amaranth/latest/stdlib.html) admits the standard library contains things which "will [be] used by essentially all idiomatic Amaranth code". When things remain in the language, it is usually because they are necessary to implement the standard library itself. The utils.py functions are very helpful, but are not *necessary* in this way.

**Why `math`?**

`utils` is, in general, not a very good name for a module because it is so broad that  in principle anything could go into it. By coincidence, all three of the methods in `utils` are some sort of math operation (more ambiguously so in the case of `bits_for`) as they are all applications of the base-2 log.

Giving the module a specific name ("math") discourages it becoming a bin for general functions which more probably ought to get their own modules. It also possibly encourages writing more math functions (possibly good).

**Could we do nothing?**

We could leave `utils` where it is and simply *define* it as being part of the language or part of the standard library, without changing its `import` location. The cost of this is the documentation would no longer be able to make definitive statements like "symbols that are part of the language are in the `.hdl` module" or "symbols that are part of the standard library are in the `.lib` module"

## Prior art
[prior-art]: #prior-art

Migen exposes a `migen.util.misc`, containing a GCD calculator, a flattening iterator, and a third function I don't understand. I think this is an example of what to avoid.

LiteX has avoided doing anything of this sort, but does have a `litex.tools` module which contains all command-line-callable scripts (see "future possibilities" below). 

## Unresolved questions
[unresolved-questions]: #unresolved-questions

Are there any "general" symbols (for example `Final` in `_utils.py`) which might in future become part of the public API, which we would be denying an "obvious" home to if we choose a finer-grained name like `math`? (And would that be a good or bad thing?)

## Future possibilities
[future-possibilities]: #future-possibilities

`utils` is the most egregious breakage of the rules about "where things live", but not the only one. Should any others be addressed also?

* There are top-level public files named `asserts`, `cli`, `rpc`, and `tracer`. `asserts` republishes symbols from `.hdl` and can be seen as a convenience, similar to the "prelude". `cli` and `rpc` make sense top level because they are command-line-callable scripts-- although if we add many more top level scripts, we may want to consider a dedicated `tools` module such as LiteX has. But what is `tracer`?
* Currently, Array lives in the language and Struct in the stdlib. That's slightly inconsistent. Could Array be moved from the language to the Stdlib? Should it?
