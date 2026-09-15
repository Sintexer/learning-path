It is guaranteed that messages will be delivered and there won't be delivery duplications.

The message is processed **and its effect applied exactly one time**, no duplicates, no loss — semantically.

**true "exactly-once" transport, in the general distributed systems sense, is essentially unachievable** — you can't have a network call plus a state change be perfectly atomic across two independent systems without some coordination cost. 

What real systems do instead is: 

**At-least-once delivery + idempotent processing = "exactly-once effect."**

In other words: let [[Kafka]] redeliver duplicates freely (that's fine, that's just [[At-least-once delivery]]), but design your consumer logic (using [[Version Guard|version guards]], [[Idempotancy Key|unique operation ID]]s, or naturally idempotent operationsc) so that processing the same message twice produces the **same end state** as processing it once. The _transport_ is at-least-once; the _outcome_ behaves like exactly-once. This is the practical, industry-standard way almost every "exactly-once" system claim actually works under the hood.