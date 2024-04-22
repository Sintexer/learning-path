
## ⌨️ `get-object-attributes`

Retrieves all the metadata from an object without returning the object itself. This operation is useful if you’re interested only in an object’s metadata.


## ⌨️ `list-object-versions`

Returns metadata about all versions of the objects in a bucket. You can also use request parameters as selection criteria to return metadata about a subset of all the object versions.

**Note:** This operation is not supported by directory buckets.

```
list-object-versions --bucket <value>
```

## ⌨️ `delete-object`

Removes an object from a bucket. The behavior depends on the bucket’s versioning state:

- If bucket versioning is not enabled, the operation permanently deletes the object.
- If bucket versioning is enabled, the operation inserts a delete marker, which becomes the current version of the object. To permanently delete an object in a versioned bucket, you must include the object’s `versionId` in the request. For more information about versioning-enabled buckets, see [Deleting object versions from a versioning-enabled bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DeletingObjectVersions.html) .
- If bucket versioning is suspended, the operation removes the object that has a null `versionId` , if there is one, and inserts a delete marker that becomes the current version of the object. If there isn’t an object with a null `versionId` , and all versions of the object have a `versionId` , Amazon S3 does not remove the object and only inserts a delete marker. To permanently delete an object that has a `versionId` , you must include the object’s `versionId` in the request. For more information about versioning-suspended buckets, see [Deleting objects from versioning-suspended buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DeletingObjectsfromVersioningSuspendedBuckets.html) .

**Note:** For restoring deleted file pass the `--version-id {deleteMarketVerionsId}` . See [[AWS S3 Bucket versioning]]

```
delete-object --bucket <value> --key <value>
```

## Optional parameters

```
[--delimiter <value>]
[--encoding-type <value>]
[--prefix <value>]
[--expected-bucket-owner <value>]
[--request-payer <value>]
[--optional-object-attributes <value>]
[--cli-input-json | --cli-input-yaml]
[--starting-token <value>]
[--page-size <value>]
[--max-items <value>]
[--generate-cli-skeleton <value>]
[--debug]
[--endpoint-url <value>]
[--no-verify-ssl]
[--no-paginate]
[--output <value>]
[--query <value>]
[--profile <value>]
[--region <value>]
[--version <value>]
[--color <value>]
[--no-sign-request]
[--ca-bundle <value>]
[--cli-read-timeout <value>]
[--cli-connect-timeout <value>]
[--cli-binary-format <value>]
[--no-cli-pager]
[--cli-auto-prompt]
[--no-cli-auto-prompt]
```