Service examples: [[AWS EFS]], [[AWS FSx]].

Files are organized in a tree-like hierarchy that consist of folders and subfolders (like in Windows File Explorer or Finder on MacOS). For example, if you have hundreds of cat photos on your laptop, you might want to create a folder called **Cat photos**, and place the images inside that folder to organize them. Because you know that these images will be used in an application, you might want to place the **Cat photos** folder inside another folder called **Application files**.

> **File storage** systems are often supported with a network-attached storage (NAS) server.

Each file has metadata such as file name, file size, and the date the file was created. The file also has a path, for example, `computer/Application_files/Cat_photos/cats-03.png`. When you need to retrieve a file, your system can use the path to find it in the file hierarchy.  
  
File storage is ideal when you require centralized access to files that must be easily shared and managed by multiple host computers. Typically, this storage is mounted onto multiple hosts, and requires file locking and integration with existing file system communication protocols.

**Use cases (really vague):**

- Web serving (path to html file)
- Analytics - Many analytics workloads interact with data through a file interface and rely on features such as file lock or writing to portions of a file. Cloud-based file storage supports common file-level protocols and has the ability to scale capacity and performance. Therefore, file storage can be conveniently integrated into analytics workflows.
- Media and entertainment - Many businesses use a hybrid cloud deployment and need standardized access using file system protocols (NFS or SMB) or concurrent protocol access
- Home directories - Businesses wanting to take advantage of the scalability and cost benefits of the cloud are extending access to home directories for many of their users. Cloud file storage systems adhere to common file-level protocols and standard permissions models. Therefore, customers can lift and shift applications that need this capability to the cloud.