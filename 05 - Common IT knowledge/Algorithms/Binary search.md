Might be seen not only in arrays, but also in trees, and math problems like finding the root of a equation.

## Array binary search implementation

- Use two pointers for L and R.
- Use while loop `while (l <= r)`, "<=" ensures you'll check both l and r elements (bounds) if a search narrows to a single element. Sometimes "<" is used, e.g. for finding a place to insert a new item. However, a different index update strategy should be used then.
- l and r are updated by `i + 1`, or `i - 1` based on the current element value compared to the desired value