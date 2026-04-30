I talked a bit with Gemini about the design of Blitz today:

One big idea is to abandon the stack machine model and switch to a binary encoding of an AST.  This makes the interpreter a little more complex, but significantly decreases the complexity of the minimum viable toolchain (for generating, printing, and debugging Blitz code), and code size is still decent.

It also simplifies the checker and makes for a cleaner prototype impl, since the model is just a really simple language.

The second big idea is to adopt Rusts MIR model wholesale.  I am a little skeptical, but Gemini claims that the subset of this logic that applies to the blitz model is actually quite simple.  In particular, Blitz kernels are monomorphized and defunctionalized, and uses tail calls instead of loops, so most of the tricky cases are already gone.

The third idea is to add back static struct/union types instead of my previous simplification to just ownership semantics on dynamic values.

The fourth idea is to have statically checked bounds checking in the type system to avoid bounds checking overhead, and to avoid needing complex bounds checking elimination in the runtimes optimizer.

In particular, this approach eliminates all laziness overhead in Blitz kernels, most bounds checking overhead, and makes unboxed data possible.

I need to evaluate these ideas more precisely, but this seems pretty plausible, and if it works out the perf sensitive bits get written in something that looks a lot like a safe subset of rust, but with zero marshalling overhead at the boundary in and out of PLAN code.