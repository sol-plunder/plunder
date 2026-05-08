In the original Plunder design, There was a guarantee that any any two pins that were equal were also pointer equal. 

While this is indeed a very convenient property, it turns out to be prohibitively expensive.

Furthermore, it also means that the runtime system needs to maintain a global data structure mapping hashes to pins, and there's a lot of questions around where that data structure lives and how it interacts with garbage collection.

The new design weakens this guarantee, and allows duplicate pins to be constructed.  Instead, user code will maintain a mapping from hashes to pins, and deduplication will only be performed in certain contexts.

# Unifying Duplicate Pins

There's some potential for allowing duplicate pins to be forwarded, so if duplicates are found they can be unified.

This is a very tricky thing to implement, because it breaks the usual guarantee that frozen pens and data on disk are always truly immutable.

however, since it is a very well behaved form of mutation (never changes the actual value, never introduces a reference from an older generation into a younger one, etc), it seems like this is possible to support.  However, the properties of this mutation, and it's interaction with all of the relevant subsystems (persist, atomicity, GC, actor processes, etc) needs to be mapped out precisely for such a feature can be committed to.