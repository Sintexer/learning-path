---
flashcards:
  q-6fpf: { hash: 2wb9crxa }
  q-5jjw: { nid: 1795259382951, hash: rzde4npy, sync: azyrp656 }
  q-aumi: { nid: 1795259383064, hash: 8de5nffv, sync: jpgw4q3h }
---
[[Leaderless Replication#Quorums for reading and writing|Quorums]] are not as fault-tolerant as they could be. A network interruption can easily cut off a client from a large number of database nodes. Although those nodes are alive, and other clients may be able to connect to them, to a client that is cut off from the database nodes, they might as well be dead. In this situation, it’s likely that fewer than w or r reachable nodes remain, so the client can no longer reach a quorum.

In a large cluster (with significantly more than n nodes) it’s likely that the client can connect to some database nodes during the network interruption, just not to the nodes that it needs to assemble a quorum for a particular value. In that case, database designers face a trade-off:
- Is it better to return errors to all requests for which we cannot reach a quorum of *w* or *r* nodes?
- Or should we accept writes anyway, and write them to some nodes that are reachable but aren’t among the n nodes on which the value usually lives? 

The latter is known as a **sloppy quorum** ::: writes and reads still require *w* and *r* successful responses, but those may include nodes that are not among the designated *n* “home” nodes for a value. By analogy, if you lock yourself out of your house, you may knock on the neighbor’s door and ask whether you may stay on their couch temporarily.
^q-6fpf

When the network interruption is fixed in a sloppy quorum, any writes that one node temporarily accepted on behalf of another node are sent to the appropriate "home" nodes. ::: This is called *hinted handoff*
^q-5jjw

Sloppy quorums are particularly useful for increasing write availability: ==as long as any w nodes are available, the database can accept writes==. However, this means that even when *w + r > n*, you cannot be sure to read the latest value for a key, because the latest value may have been temporarily written to some nodes outside of *n*.
^q-aumi