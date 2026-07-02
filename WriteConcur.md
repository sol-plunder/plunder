The finer grained transactionality rules in Shrine seem like they open up a lot of potential for more efficient snapshotting.

Because separate subtrees represent separate transactionality universes, their new states can be committed concurrently (and in parallel).

Our persistence system seems like it composes perfectly with this, since we support multiple independent non-commiting writes at the same time, and only need to synchronize when updating the root (namespace state).

This seems like an improvement over the strict [CogDrone](CogDrone.md) model which is strictly linear.