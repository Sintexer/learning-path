In contrast to the [[Forward Compatibility]], means newer code can read and modify data that was written by older code

Backward compatibility is normally not hard to achieve: as author of the newer code, you know the format of data written by older code, and so you can explicitly handle it (if necessary by simply keeping the old code to read the old data).

In terms of how [[Data Encoding#Binary formats]] support backward compatibility, 
- as long as each field has a unique tag number, new code can always read old data, because the tag numbers still have the same meaning. 
- The only detail is that if you add a new field, you cannot make it required. If you were to add a field and make it required, that check would fail if new code read data written by old code, because the old code will not have written the new field that you added. Therefore, to maintain backward compatibility, every field you add after the initial deployment of the schema must be optional or have a default value.