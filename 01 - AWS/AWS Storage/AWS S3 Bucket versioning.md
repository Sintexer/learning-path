AWS bucket versioning is a feature in [[AWS S3]] that allows you to keep multiple **versions** of an object in the same bucket. With versioning, you can preserve, retrieve, and restore every version of every object in your bucket. This makes it easy to **recover** from both unintended user actions and application failures.

Once you enable versioning for a bucket, S3 automatically generates a unique **version ID** for the object each time it is uploaded. If you upload another object with the same key, a new version of the object is created in the bucket, instead of overwriting the existing object.

> [!warning] Costs are increasing with versioning
> Storage costs are incurred for all objects in your bucket, including all versions. To reduce your Amazon S3 bill, you might want to delete previous versions of your objects when they are no longer needed.
## Delete marker

If you delete an object without specifying a **version ID**, [[AWS S3|S3]] adds a **delete marker**, making the object invisible by default. However, all previous versions remain in the bucket. If you delete an object's specific version, it just deletes that version of the object; other versions and the delete marker (if any) remain intact.

## Bucket versioning states

After you version-enable a bucket, it can never return to an **unversioned** state. But you can _suspend_ versioning on that bucket. The versioning state applies to all of the objects in that bucket. When you enable versioning in a bucket, all new objects are versioned and given a unique version ID. Objects that already existed in the bucket at the time versioning was enabled will thereafter _always_ be versioned and given a unique version ID when they are modified by future requests.

Buckets can be in one of three states:

- **Unversioned** (the default)
- **Versioning-enabled**
- **Versioning-suspended**
