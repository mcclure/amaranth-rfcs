- Start Date: (fill me in with today's date, 2024-02-12)
- RFC PR: [amaranth-lang/rfcs#0000](https://github.com/amaranth-lang/rfcs/pull/0000)
- Amaranth Issue: [amaranth-lang/amaranth#0000](https://github.com/amaranth-lang/amaranth/issues/0000)

# Consolidate d, domain, domains

## Summary
[summary]: #summary

Merge the `domain` and `domains` fields of Module into a single object. Deprecate the `domain` accessor so that `.d` and `.domains` are the only accessors.

## Motivation
[motivation]: #motivation

This came up as part of a project to document Amaranth's undocumented features. Currently module objects have three fields named `d`, `domain`, and `domains`, with `d` being an alias for `domain`. The documentation in places mentions both `d` and `domains`, but does not currently explain the difference, and currently does not document `domain` in any way. In practice, both `domain` and `domains` provide access to the module's synchronous clock domains, but `domain` is "read-only" and `domains` is "write-only"; you can say `m.d.sync += ...` or `m.domains["sync"] =` but the reverse statements are not allowed. None of this seems to have been intentional or planned in any way.

This is confusing, difficult to document, and probably not well understood by most Amaranth users. However, because the two objects' uses are "non-overlapping", they could be merged into one object without drawbacks.

## Guide-level explanation
[guide-level-explanation]: #guide-level-explanation

"[Clock domains](https://amaranth-lang.org/docs/amaranth/latest/guide.html#clock-domains)", after introducing `m.domains` and before the first Note:

> For convenience, `m.domains` is also available at the alias `m.d`, which has been used throughout this guide. The recommended style is to call `domains` by its full name when adding new values to the collection.

This text is sufficient to make the new behavior clear to new users. Existing users will not find their intuition or code needs to be adjusted, as the new "recommended style" matches the old behavior.

## Reference-level explanation
[reference-level-explanation]: #reference-level-explanation

This would naturally appear as part of the documentation of the `Module` class, which is not yet written.

Note from Catherine:

> what you do not list in the RFC is how to implement `m.domains["foo"]` += considering that it would return a `ClockDomain` object on which then `__iadd__` would be called
>
> `[]+=` is not a unified operation in Python
>
> or I guess it's the other way around, how would it return a ClockDomain object if m.domains["foo"] currently returns a proxy object implementing +=

## Drawbacks
[drawbacks]: #drawbacks

If `domain` is being explicitly used in existing Amaranth code this code would break (unless we follow "alternative 1" below); however it's possible no such code exists, since `domain` was never documented.

## Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

Alternatives include

1. Leave `domain` in place; have `m.d`, `m.domain`, and `m.d.domains` all be aliases for one object.
2. Do nothing.

The downsides of both these options is they will significantly complicate explaining these features in the documentation; and probably, if a feature is too complex to easily document, this is a sign the feature itself may be flawed. Alternative 1 would have a lesser impact on documentation clarity because we could use the language "a second alias, `m.domain`, is also allowed for legacy reasons, but discouraged".

## Migration path

For 1 release, the `.domain` property should become an alias for `m.d.domains`, but when accessed it should print a "this property is deprecated, use m.domains" warning.

## Prior art
[prior-art]: #prior-art

[Migen has](https://gist.github.com/cr1901/5de5b276fca539b66fe7f4493a5bfe7d) a paired syntax somewhat similar to Amaranth's current design: There's a `self.clock_domains` field assigned to for declaring new clock domains, broadly similar to amaranth `m.domains`; and domains previously assigned to `clock_domains` are read back as properties of `self.sync`, similar to the properties of `m.d` (aka `m.domain`).

## Unresolved questions
[unresolved-questions]: #unresolved-questions

- Does code using the `m.domain` field exist in the wild?

## Future possibilities
[future-possibilities]: #future-possibilities

*n/a*
