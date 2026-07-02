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

----

How do you do this?
This data extension is interesting. You could imagine providing ST in this way, possibly as a pure mock-sublanguage, but accelerated.

Do you want crash or cast to 0?
My instinct is to make it crash (or even have undefined behavior).  The idea is that is should be available *only* in outer shell code, and never in stored data or in virtualized contexts.

Indeed, you could support mvars, tvars, and all that!  However, this doesn't work well with our GC model, since it means that threads can share state.

Stref would work, but all of this stuff only works in a limited "outside shell" context, and not in any pure context, so you can't actually use it for much.
You don't need the data extension to provide st within...

Oh, interesting.
Like, *also* extend the language of mock with handles?  I had not considered that.
My intuition is that this would run into problems, but if it doesn't, that would actually solve a lot of design issues.
Yeah. You really need stref for unification.
We can do stref now by just having a handle as a nat.
And you want your compiler to be pure and hostable in an inner layer
It’s probably too slow
No it isn't.
This is equivalent to “explicit substitutions” and iirc no real systems use bc of speed
Stref as nat costs a bounds check and an indirection, tiny constant time cost.
Uhh. You’re imagining a dense array of refs?
Yes
These are churned through really fast, pretty much one per type tree node in practice, and some survive. So nursury-gc-like
Big win of handles is that they are visible to GC.  Big downside is that they are super limited.  Cannot serialize them, cannot freeze them, cannot put them in data structure which use pins, etc.
You want to “zonk” any type that has refs in it anyway before you do those things
Yeah, but it there is a difference between "you don't want to do that anyways" and "the operations of the system no longer compose"
Right
What happens when an st ref is returned from stmock? In hs, the rank-2 type of runST stops you, but here…
So, you need a very strong argument that this fragile non-composable stuff pays for itself.

If you *do* have handles, you can always implement the table yourself.. yeah, just need to think through every possible use case and all possible things you might want to do with it.

A promising direction, but will take weeks or months to evaluate.
Right
Yeah, that is a problem.
Oh!
No it isn't.
We have to force the result anyways.  Evaluating or forcing a handle is a crash
I suspect there’s a general, composable solution for “data model extension mocks” here
Handles would essentially be treated like black holes, formally.
If you ever touch it at all, the whole world collapses in on itself.
Or you could arbitrarily say that they zonk through on exit?
But you touch it to read and write?
In hs, you can have a thunk that yields an stref
No, you pass it as an argument to a special effect.
Right!
Neat
So it has “special eliminators”
Yeah, stread / stwrite
Otherwise it's like a black hole.
The mock exit problem?
There is no problem, because mock exit always normalized, so you just get a crash.
It is?
It has to be.  It's semantically nonsensical to try to return a thunk from mock.
A thunk itself is a handle into internal mock evaluation state.
Formally
I think this might be wrong
Formally from the outside the inner thunk is a fully saturated inner recursive call to the eval func
Well if you think about it, you'll find that it is very, very fundamentally right, and non arbitrary.
Which is how you’r formally implement laziness internally in mock anyway
Mock returns a plan value, not a plan value and some sort of evaluator state.
So, formally, there's no way to encode what you are imagining.

And informally, allowing mock to return thunks, would mean that every thunk would need to be tagged with an evaluation context, which would add incredible amount of overhead.
Also if you think about it, sealed evaluations can have effects. 

So if you were to return a thunk, your concept of the ordering and interpretation of these effects becomes deeply entwined with the semantics of the outside evaluator.
This idea of encoding handles as black holes, in both the outermost context, and in mock is pretty compelling, needs to be explored in detail, but this seems really promising.
It's interesting how the extreme focus on purity and composability has led to designs which are ultimately able to reintroduce normal impure ideas in limited contexts, without compromising all of the advantages of purity.

For a long time we had no effects at all, and we got a lot of advantages but also perf issues.

Those were solved by reintroducing effects through the syscall system, this reintroduces impurity, but in a way which does not break the hard purity in other context, and that hard purity is essential to all of the other major architectural wins.

This handle is the same idea.  We ruthlessly eliminate any type of "handle into runtime state" because it breaks code mobility and permenence.

But it seems like we can maybe reintroduce that idea without breaking everything by just also accepting some weird restrictions around that (can't evaluate them).

The resulting system has a bunch of seemingly arbitrary, and highly counter intuitive quirks, but those quirks keep these impure ideas from contaminating the design, which is the secret to all of our unique features:

effects mean different things in different contexts, you have to know what context you are in.

Catching exceptions forces you to normalize everything on input, and again on output.

You cannot evaluate handles.

Normalizing a function normalizes all referenced data, in practice you need to avoid functions which close over handles or thunks.

All of these restrictions seem very weird, but the result is a frozen model, orthogonal persistence, pausless gc, sendable closures, the ability to publish data which includes closures online, etc.
Actually!  I bet this restriction could also maybe solve the threaded GC issue?

Messages between actors are normalized, so you can't send handles.  You need a special send-with-handle operation to avoid evaluating the handle.

And then, when we are in the collecting roots phase, we can have that operation have a write barrier.

We start root collection with a queue of threads, which is initially just the root thread.  Whenever we're collecting roots from a thread, if we see a handle to a thread that hasn't been collected yet, we add it to this queue. 

If a thread sends a handle to another thread, where the destination is known to be live, and the handle referenced thread isn't, then we add that to the queue.

Once the queue is empty, all unmarked threads can be freed.