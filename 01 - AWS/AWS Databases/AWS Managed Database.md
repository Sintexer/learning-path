> See [[AWS Managed Service]]

To shift more of the work to AWS, you can use a **managed database service** instead. These services provide the setup of both the [[EC2]] instance and the [[Database|database]]. Unlike the [[AWS Unmanaged Database]], you don't need to worry about scaling, high availability, backups or security patches.

However, in this model, you’re still responsible for database tuning, query optimization, and ensuring that your customer data is secure. This option provides the ultimate convenience but the least amount of control compared to the two previous options.

![[AWS Managed Database diagram.png]]