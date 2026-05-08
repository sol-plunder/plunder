Mega pins are pinned pins.  The runtime system maintaines a reachability cache for each megapin.

A megapin has an adjacent list of all contained megapins, and the set of all normal pins inside of it (up to, but not including the next megapin).

For example:

```
A = <<[B D]>>
B = <[7 7]>
C = <[6 6]>
D = <<[C C]>>
```

Here, A is a megapin containing B, and linking to D.

D is a megapin containing C and linking to nothing.

The purpose of megapins is to make mark traversals of massive heaps much cheaper.  This is especially important on disk, where seeks are expensive.
When a pin is first constructed, it is just a simple data structure record on the per actor heap.  The item inside of it is not normalized, the data isn't compacted, the cryptographic hash has not yet been calculated, and it is still local to the current thread.  Constructing a liquid pin is very cheap.

When a pin is frozen (or sometimes called jailed), it is normalized, compacted, and hashed.

Only frozen pins can be persisted, and only frozen pins can be shared between threads.

You can think of freezing a pin as an explicit operation which moves a pin into the second GC generation.