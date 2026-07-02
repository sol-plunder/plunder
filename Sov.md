> https://acko.net/blog/on-variance-and-extensibility/

I think my reaction to all of this is that vendor lock-in is just the only thing that actually works, any attempt to do anything else always devolves back to that eventually, with a whole lot of extra mess that comes out of the attempt to avoid it.

Because of that, I don't think types actually make sense in a system with multiple independent authors of code.  Types are proofs of the relationships between components in one coherent system created by one entity.

As soon as you have multiple sovereigns, the best that you can do is to negotiate protocols, pairwise between them, and accept the reality that these protocols are not enforceable.

> For a given purpose, you need a well-defined single type T that sets the ground rules for both data and code, which means T must be a common language.

That's PLAN!

My thinking is that PLAN should be a lingua Franca between systems, and types should be used within systems, but not between them.  Between them, you just use PLAN like JSON, a standardized container format, but with no guarantees about the shape of what is within.