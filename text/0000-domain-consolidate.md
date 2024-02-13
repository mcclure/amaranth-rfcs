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

## Drawbacks
[drawbacks]: #drawbacks

If `domain` is being used in existing Amaranth code— the `domain` language is taken from migen, so although `domain` has never been documented, ported code may contain it anyway— this code would break.

## Rationale and alternatives
[rationale-and-alternatives]: #rationale-and-alternatives

Alternatives include

1. Leave `domain` in place; have `m.d`, `m.domain`, and `m.d.domains` all be aliases for one object.
2. Do nothing.

The downsides of both these options is they will significantly complicate explaining these features in the documentation; and probably, if a feature is too complex to easily document, this is a sign the feature itself may be flawed. Alternative 1 would have a lesser impact on documentation clarity because we could use the language "a second alias, `m.domain`, is also allowed for legacy reasons, but discouraged".

## Prior art
[prior-art]: #prior-art

Migen contains the `domain` field, behaving largely the same way as amaranth `m.d`/`m.domain`. The `m.domains` syntax is new to Amaranth.

## Unresolved questions
[unresolved-questions]: #unresolved-questions

- Does code using the `m.domain` field exist in the wild?
- Does the "style guide" of existing Amaranth API use the singular or plural when exposing collections as fields? (I.E. is it `m.domain` or `m.domains` which better matches similar places in the API?)

## Future possibilities
[future-possibilities]: #future-possibilities

*n/a*
