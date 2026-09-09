The most widely used [[Database Index|indexing structure]].

B-trees keep key-value pairs sorted by key, which allows efficient key-value lookups and range queries. It is designed to store sorted data and allow searches, sequential access, insertions, and deletions in logarithmic time $(O(log⁡n))$.

B-trees break the database down into fixed-size blocks or pages, traditionally 4 KB in size (sometimes bigger), and read or write one page at a time. This design corresponds more closely to the underlying hardware, as disks are also arranged in fixed-size blocks. Each page can be identified using an address or location, which allows one page to refer to another—similar to a pointer, but on disk instead of in memory.

Unlike binary search trees (where each node has a maximum of two children), B-Tree nodes can have a **large number of children** (often hundreds or thousands). This specific design makes B-Trees the industry standard for databases and file systems storing data on block-based storage (HDDs and SSDs).

The number of references to child pages in one page of the B-tree is called the **branching factor**.

## Why B-Tree

Computers read and write data from disk in blocks or **pages** (typically 4KB to 16KB in size). Fetching a 16KB page from an SSD or hard drive takes vastly more time than reading data from RAM.

If you used a standard binary search tree ([[Binary Search Tree]] or [[AVL Tree]]) on disk:
- Each node would typically store only a single key and two pointers.
- Traversing the tree would require a separate disk read for almost every single node level you visit, leading to excessive disk seeks (high I/O cost).

**B-Trees solve this by maximizing "branching factor."** By packing many keys into a single disk page, a single disk read gives the database a huge chunk of routing information, drastically reducing the number of disk accesses required to find data.

## B-Tree anatomy

A B-Tree consists of:
- **Root Node:** The entry point of the tree.
- **Internal Nodes:** Nodes that contain keys used for routing (acting as signposts) and pointers to child nodes. They do not store the actual data rows.
- **Leaf Nodes:** Nodes at the very bottom of the tree that contain the actual keys (and sometimes data pointers or data rows).

Key Characteristics of B-Trees:
1. **Self-Balancing:** B-Trees automatically grow and shrink. All leaf nodes reside at the exact same depth, ensuring that every search takes the same predictable number of steps.
2. **High Fan-Out (Branching Factor):** Because nodes are large (sized to match disk pages), a node can have dozens or hundreds of children. This keeps the tree very "short" (usually only 3 to 4 levels deep even for hundreds of millions of rows), meaning you only need 3 or 4 disk reads to find any record.
3. **Sorted Keys:** Keys within each node are always stored in sorted order, allowing binary search within the node.

## B+Tree

While the classic academic "B-Tree" stores keys and data pointers in both internal and leaf nodes, **relational databases almost exclusively use a variant called the B+Tree**.

In a **B+Tree**:
1. **Data resides only in the leaves:** Internal nodes store _only_ copy keys and child pointers used for navigation. They do not store data values or row pointers. This allows internal nodes to pack even more keys, increasing the branching factor further.
2. **Linked Leaf Nodes:** All leaf nodes are connected sequentially in a doubly-linked list.

### Why B+Trees Dominate Databases:

- **Efficient Range Queries:** If you run a query like `WHERE age BETWEEN 20 AND 30`, a B+Tree finds the starting key (say, age 20) via tree traversal, and then simply **walks horizontally through the linked leaf nodes**. In a standard B-Tree or Hash index, range queries require expensive zig-zagging up and down the tree or full scans.
- **Predictable Read Performance:** Every point lookup traverses the exact same number of levels from root to leaf.

## Operations in B-Tree / B+Tree

**Search (Read):**
1. Start at the root node.
2. Perform a binary search within the node to find which child pointer to follow.
3. Repeat until you reach the leaf node.
4. Time complexity: O(log⁡n) disk reads. Because the branching factor is so high, log⁡bn results in a very small number (typically 3–4 lookups).

**Insertion (Write):**
1. Traverse down to the appropriate leaf node and insert the key in sorted order.
2. If the leaf node exceeds its capacity (overflows), it **splits** in half, and the middle key is pushed up to the parent node.
3. If the parent node overflows, it splits too, potentially cascading all the way up to the root (which can create a new root and increase the tree's height).

**Deletion:**
1. Locate and remove the key from the leaf node.
2. If the node falls below minimum capacity (underflows), it borrows keys from a sibling node or **merges** with an adjacent node.

## Making B-Tree reliable

Unlike [[LSM Tree]], which never modify files in place, but rather only append to a memtable, B-Tree has to write to the disk. It rewrites one block at a time. And as it modifies block in place, all references to this block remain valid. However insert and deletion might grow or shrink the segment. So that raises a question: what will happen if the database crashes in the middle of such non-atomic operations? To restore after a crash databases use the [[WAL]].

An additional complication of updating pages in place is that careful concurrency control is required if multiple threads are going to access the B-tree at the same time — otherwise a thread may see the tree in an inconsistent state. This is typically done by protecting the tree’s data structures with [[Database Latch|latches]] (lightweight locks). [[LSM Tree|Log-structured approaches]] are simpler in this regard, because they do all the merging in the background without interfering with incoming queries and atomically swap old segments for new segments from time to time.

## Optimizations

As B-trees have been around for so long, it’s not surprising that many optimizations
have been developed over the years. To mention just a few:

- Instead of overwriting pages and maintaining a WAL for crash recovery, some databases (like LMDB) use a copy-on-write scheme. A modified page is written to a different location, and a new version of the parent pages in the tree is created, pointing at the new location. This approach is also useful for concurrency control.
- Store pointers to sibling nodes
- We can save space in pages by not storing the entire key, but abbreviating it. Especially in pages on the interior of the tree, keys only need to provide enough information to act as boundaries between key ranges. Packing more keys into a page allows the tree to have a higher branching factor, and thus fewer levels
