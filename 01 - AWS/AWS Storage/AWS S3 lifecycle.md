If you keep manually changing your [[AWS S3 Bucket#Object|objects]], such as your employee photos, from storage tier to [[AWS S3 storage classes|storage tier]], you might want to automate the process by configuring their Amazon S3 lifecycle. When you define a lifecycle configuration for an object or group of objects, you can choose to automate between two types of actions: **transition** and **expiration**.

- **Transition actions** define when objects should transition to another storage class.
- **Expiration actions** define when objects expire and should be permanently deleted.

For example, you might transition objects to **S3 Standard-IA** storage class 30 days after you create them. Or you might archive objects to the **S3 Glacier Deep Archive** storage class 1 year after creating them.

The following use cases are good candidates for the use of lifecycle configuration rules:

- **Periodic logs:** If you upload periodic logs to a bucket, your application might need them for a week or a month. After that, you might want to delete them.
- **Data that changes in access frequency:** Some documents are frequently accessed for a limited period of time. After that, they are infrequently accessed. At some point, you might not need real-time access to them. But your organization or regulations might require you to archive them for a specific period. After that, you can delete them.