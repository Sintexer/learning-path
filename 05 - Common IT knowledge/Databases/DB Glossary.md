## Table Heap

A table heap is an array of 8KB disk pages. A B-tree leaf node stores (Key Value, [[#TID]]).  The leaf says: _"Go to Page #542, and read slot #4."_

## TID

Tuple ID (Row ID). A `TID` is literally `(Page Number, Item Offset)`.

## Cardinality

The number of unique values in a column.

## Selectivity

How effectively a value narrows down the search space. High selectivity means a search matches very few rows (e.g., UUIDs have extremely high selectivity; `is_active` boolean has low selectivity).