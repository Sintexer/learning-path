Named after ship compartments — if one compartment floods, bulkheads (physical walls) stop the whole ship from sinking.

**Applied to software:** give each downstream dependency **its own isolated pool** of threads/connections. If Service B is slow and eating up all its allocated threads, that doesn't touch the threads reserved for calling Service C — unrelated requests stay healthy.