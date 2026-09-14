## Generalized Search Tree

GiST is not a single index structure; it is an **extensible, balanced tree framework** designed for non-scalar, multi-dimensional, or hierarchical data.  
It uses the concept of **bounding boxes/predicates**. An internal node describes a property that holds true for all nodes below it (e.g., "all geometric shapes in this branch fall within Box (X1,Y1,X2,Y2)").

## Core Use Cases:

1. **Geometric / Spatial Data (PostGIS):** Nearest neighbor queries, "points within polygon", boundary overlaps (`&&`).
2. **Range Types:** Checking for overlapping date ranges (`daterange`, `tsrange`) using operators like `&&` (e.g., preventing double-booking rooms).
3. **K-Nearest Neighbors (k-NN):** Finding the K closest points or vectors using distance operators (`<->`).

## GIN vs. GiST (The Classic Interview Comparison):

Related to [[GIN Index]].

|Feature|GIN|GiST|
|---|---|---|
|**Primary Structure**|Inverted Index (Item → Rows)|Balanced Search Tree (Hierarchical Bounding)|
|**Best For**|Many identical keys across rows (Arrays, Text, JSONB)|Spatial, Range types, Nearest Neighbor (`<->`)|
|**Search Speed**|Faster for static text/array searches|Slower (lossy traversals require re-checks)|
|**Build & Update Speed**|Very slow writes / high write amplification|Faster dynamic updates / lower write overhead|
