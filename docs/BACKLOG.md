# Backlog

## Unordered case notation in comm search

`src/comm-search/search3.js` matches search terms against message content with SQLite
`GLOB` patterns built from the term exactly as typed. That is correct for cycles, where
sticker order carries meaning, but wrong for cases where the notation names an unordered
set of pieces.

The two-edge flip was the first instance found: `BL BR flip` and `BR BL flip` name the
same case but returned different results. That specific ordering bug is fixed by the
`EdgeFlip` term type, which queries both orderings.

The general problem is not fixed. Remaining work, roughly in order of value:

### Sticker-face equivalence for flips

A flip happens in place, so `UF DF flip` and `FU DF flip` name the same physical case -
either sticker of an edge identifies the same piece, and one alg serves all spellings.
Only the exact written sticker matches today. With ordering already handled that leaves
four spellings per pair, of which two are found.

Fixing it means deciding how far to canonicalize. Options include normalizing each
sticker to a canonical face at query time and expanding to all spellings in the glob
list, or normalizing message content at index time in `handleReadComms`
(`src/comm-search/search.js`) so the stored form is canonical and only the query needs
rewriting. The second is cheaper per query but changes what is stored.

### Audit other case types for unordered notation

Flips are unlikely to be the only unordered notation in the channel. Corner twists in
particular are worth checking - if two-corner twists are posted, they have the same
property. Any case type where the notation names a set rather than a cycle needs the
same treatment.

### Consider a shared mechanism

If more than one unordered case type turns up, the per-type `this.glob = [a, b]` approach
should probably become a shared helper that takes the unordered stickers and emits every
permutation, rather than each term type spelling its orderings out by hand. Not worth
building for a single case type - revisit once there is a second.
