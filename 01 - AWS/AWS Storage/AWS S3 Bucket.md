**Bucket** is the fundamental container for objects in [[AWS S3]]. Buckets must have **globally unique name** and they are defined at the **[[Region|region]] level**. When you choose a bucket name, AWS stops you from choosing a bucket name that has already been chosen by someone else in another AWS account. However, name is released on bucket deletion.

S3 identifies objects in part by using the object name. For example, when you upload an employee photo to Amazon S3, you might name the object _employee.jpg_ and store it in a bucket called employees. Without [[AWS S3 Bucket versioning]], every time you upload an object called employee.jpg to the employees bucket, it will overwrite the original object. If you want, you can enable **versioning**, and your files will **delete softly** and you'll have access to all file **versions**. But you must be careful with this feature as it can't be fully turned-off after first enabled.

**Buckets tied to the regions**. You might choose bucket region to optimize latency, minimize costs or address regulatory requirements.

Buckets serve several purposes:

1. They organize the **S3 namespace** at the highest level.
2. They **identify the account** responsible for storage and data transfer charges.
3. They play role in **access control**.
4. They serve as the unit of **aggregation** for usage reporting.

## Bucket naming

The following are some examples of the rules that apply for naming buckets in **S3**. For a full list of rules, see [the link](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html). This rules are important since bucket name is used in object URLs.

- Bucket names must be between 3 (min) and 63 (max) characters long.
- Bucket names can consist only of lowercase letters, numbers, dots (.), and hyphens (-).
- Bucket names must begin and end with a letter or number.
- Buckets must not be formatted as an IP address.
- A bucket name cannot be used by another AWS account in the same partition until the bucket is deleted.

> [!tip]
> When naming a bucket, choose a name that is relevant to you or your business. For example, you should avoid using AWS or Amazon in your bucket name.

## Object

**Object** is uniquely identified data stored inside the bucket, and **id** consists of a **bucket name**, **key** and **version ID**. Each bucket object is **accessible by its own URL**.

The **object key** (key name) uniquely identifies the object in an [[AWS S3|S3]] bucket. When you create an object, you specify the key name. As described earlier, the **S3** model is a flat structure, meaning there is **no hierarchy of subbuckets or subfolders**. However, the Amazon S3 *console does support the concept of folders*. By using key name prefixes and delimiters, you can imply a logical hierarchy.

> [!example]
> Suppose your bucket called _testbucket_ has two objects with the following object keys: `2022-03-01/AmazonS3.html` and `2022-03-01/Cats.jpg`. The console uses the key name prefix, `2022-03-01`, and delimiter (`/`) to present a folder structure
>
> Example URL: `http://testbucket.s3.amazonaws.con/2022-03-01/AmazonS3.html`

Object consists of **key**, **version ID**, **value**, **metadata**, **sub-resources** and **access control information**.

