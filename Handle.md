Hmm, I guess using wasm means that all the JS/DOM stuff gets a lot more complex.

Can't easy do the thing where you just allow any js code to be run anywhere and extend the data model with JS values (undefined behavior if you try to do PLAN stuff with them).

I wonder what the right approach is, then?
Maybe you are already doing something like this this, I only skimmed the pr, but here is my thinking about how to handle this:

- Extend the data model with JSHandle

- crash if you try to freeze this into a pin or try to run a PLAN operation against a JSHandle.

- Provide an eval effect which runs some JavaScript in the outer environment, returning a handle.

- Release the handle when unreachable.

- Evaluating js expressions, and calling js functions are the only effects.

Should be fast, minimal and fully expressive?
Similarly, I had this same idea around the actor ABI: In the reaver impl, each actor maintains an explicit file descriptor table of handles to other actors, which is quite cumbersome and complex.

Idea: just use this same approach, extend the data model with handles, and just have actor-handle as a first class datatype (which cannot be put into frozen pins and crashes in introspection).

Big open question is whether or not this has an overly onerous effect on our GC innovations.  This complicates root collection because threads can now have cyclic references, though this graph is static up to messaging.

If this can be solved well, it is probably a good idea.
Oh, also you have to make it possible to send unfrozen data between actors via a simple copy (to an ephemeral region or otherwise), so that handles can be transfered.

This is probably a good idea anyways, since it means that you don't pollute gc2 with every message by freezing a pin.