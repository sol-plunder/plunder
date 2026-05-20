> What’s the rationale behind 0 being the error value in a bunch of the BPLAN primops

For example, out of bounds indexing?

The basic thought is that the semantics need to be deterministic and simple, and that the formalism has no mechanism for signaling errors.

If you need an error to be signaled, you need to supply that yourself by wrapping the primitive (e.g. using Br instead of Ix).  And since the primitive itself is explicitly not taking responsibility for signaling errors, the error cases still need to be formalized as something and 0 is just the smallest value.
Divide by zero is another such case, I forget how that is currently defined, but it is tricky!

The hardware will signal an error, and the formal implementation says it should loop forever.

Operationally, it would be preferable for it to raise a "DivideByZero" exception, but there isn't a great way to encode that formally.

So, none of the three behaviors is acceptable on all metrics, but you can get the right operational behavior by wrapping the primitive with something that throws an exception, and then relying on optimization to eliminate the overhead.