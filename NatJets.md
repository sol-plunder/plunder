# On the Manual Implementation of Natural Number Primitives

Each PLAN runtime should roll its own bignum implementation instead of relying on one from its context (JS bignums, for example), and especially instead of using libgmp.

My thinking is that the core jetted operations should be portable, so there should not be any asymptomatic differences in performance per impl, otherwise code will only work in theory across impls, and not in practice.

The specific algorithm for each jet, the asymptomatics, and a loose impl outline should all be included in the operational spec.

GMP is *absurdly* complicated, which is awesome/necessary for some use cases, but sucks in general, so it is not a good default for this specification.

Also, GMP has licensing restrictions.  It's lgpl, so you can distribute statically linked binaries that use it.

For a real algorithms that require high performance bignum operations, we should find an alternate construction, instead of baking this into the jets.  Probably either a jetted CPU DSL that allows you write high speed kernels in a nice formal and portable way, and/or a "CPU" hardware device, that lets you generate asm blocks and run them directly on hardware for specific CPU ABIs.