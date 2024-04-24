Service examples: [[AWS S3]].

In **object storage**, files are stored as *objects*. Objects, much like files, are **treated as a single, distinct unit of data** when stored. However, unlike file storage, these objects are stored in a bucket using a flat structure, meaning there are **no folders**, directories, or complex hierarchies. Each **object contains a unique identifier**. This identifier, along with any additional **metadata**, is bundled with the data and stored. Unlike [[AWS File Storage]] or [[AWS Block Storage]], Object storage follows [[WORM]] principle.

Changing just one character in an object is more difficult than with block storage. When you want to change one character in an object, the entire object must be updated.

With object storage, you can store almost any type of data, and there is no limit to the number of objects stored, which makes it readily scalable. Object storage is generally useful when storing large or unstructured data sets.

**Use cases:**

- Data archiving
- Backup and recovery
- Rich media